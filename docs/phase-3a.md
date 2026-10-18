# Phase 3a: walling off the server (2026-10-18)

The whole point of this project: a no-name pet camera on my Wi-Fi shouldn't be
able to reach my server. Real VLANs for Wi-Fi need an access point that can tag
traffic, and my mesh router can't. So this phase gets most of the way there
with the gear I already have.

## The idea

Before, the server and every other device hung off the same switch, so traffic
between them never touched the router. Nothing could filter it.

I moved the server's cable from the switch to its own port on the MikroTik.
Now every connection to the server has to cross the router. The server keeps
the same IP, so nothing that points at it had to change.

## Allow-list, not block-list

I didn't want to hunt down every IoT device and block it one by one. New stuff
shows up all the time. So it's flipped: only my desktop and the SIEM VM get full
access. Everything else gets exactly what it needs and nothing more:

| Allowed to the server | Why |
|---|---|
| DNS (53) | Pi-hole runs on the server, everyone needs it |
| Jellyfin (8097) | The TVs stream from it |
| Tailscale direct (UDP 41641) | My phone reaches everything else through Tailscale |
| Ping | Troubleshooting |

Everything else gets logged and dropped.

```routeros
/ip firewall filter
add chain=forward action=jump jump-target=to-server dst-address=10.20.0.38 connection-state=new
add chain=to-server action=accept src-address-list=srv-admins
add chain=to-server action=accept protocol=udp dst-port=53
add chain=to-server action=accept protocol=tcp dst-port=53
add chain=to-server action=accept protocol=tcp dst-port=8097
add chain=to-server action=accept protocol=udp dst-port=41641
add chain=to-server action=accept protocol=icmp
add chain=to-server action=log log-prefix=SRV-BLOCK limit=10/1m,5:packet
add chain=to-server action=drop
```

Router admin (SSH, web, Winbox) is locked to the same short list.

## The part that almost fooled me

Turning on `use-ip-firewall` on the bridge should send bridged traffic through
the firewall. I turned it on, tested, everything worked... and every counter
said zero. The router's switch chip was still forwarding between those two
ports in hardware, so the firewall never saw a packet. "Everything still works"
was the bad sign, not the good one.

Fix: turn off hardware offload on the server's port only.

```routeros
/interface bridge settings set use-ip-firewall=yes
/interface bridge port set [find interface=ether4] hw=no
```

Counters started climbing right away. Lesson: when you add a firewall rule,
check the counters, not just that things still work.

I also dropped IPv6 to and from that port at the bridge filter, so nothing can
go around the IPv4 rules.

## A safety net

If I'd gotten a rule wrong I could have locked myself out, since I manage the
router through the server. The router's security mode blocks its scheduler, so
I used a background job that would undo the change after 3 minutes unless I
killed it first. Tests passed, killed the job.

## Testing it

From my phone, on Wi-Fi, Tailscale off:

- Server dashboard on the LAN address: spun and timed out
- Jellyfin: worked
- Internet: fine

The router log caught every attempt:

```
SRV-BLOCK to-server: in:bridge(ether2) out:bridge(ether4), connection-state:new,
proto TCP (SYN), <phone>:65146->10.20.0.38:80
```

With Tailscale on, the dashboard works through the Tailscale address.

## What this doesn't fix yet

- IoT devices on the Wi-Fi can still see phones and laptops on the same Wi-Fi.
  That traffic stays inside the mesh router and never reaches the firewall.
- No guest Wi-Fi.

Both need a VLAN-capable access point. That's Phase 3b.

## Ideas for later

Ship the router's SRV-BLOCK log to my Wazuh SIEM, so blocked attempts against
the server show up as alerts.
