# Phase 2: MikroTik becomes the router (2026-10-18)

Goal: swap the router without anyone in the house noticing. Same subnet, same
gateway address, same DNS server, same reserved addresses. The mesh router
drops to bridge mode and just does Wi-Fi.

## The problem with plugging in a new router

Out of the box the MikroTik runs a DHCP server on its LAN ports. Plug it into a
live network and it starts handing out its own addresses. And I can't give it
my real gateway address while the old router still has it, or two devices
would fight over the same IP.

## How I did it

1. **Gave the server a temporary second address** on the MikroTik's factory
   subnet, so it could talk to the router without touching anything else.
2. **Armed a watcher** on the server that pinged the router's factory address
   in a loop. The second it answered, it logged in and disabled the router's
   DHCP server. Then I plugged the MikroTik into the switch. Nothing on the
   network picked up a wrong address.
3. **Backed up the factory config** on the router before changing anything.
4. **Staged the whole config disabled:** the real LAN address, the DHCP pool,
   DNS pointing at Pi-hole, the reserved addresses for the server, my desktop
   and the SIEM VM. Also turned off telnet, FTP, the API and the router's file
   sharing.
5. **Hung a cutover script off the WAN.** The router's internet-side DHCP
   client runs a script when it gets a lease. That script turns on the LAN
   address and the DHCP server. So the switch-over happens by itself the
   moment the fiber is plugged in, even though my remote session drops when
   the internet does.
6. **The swap:** mesh router to bridge mode in its app, then the fiber cable
   from the mesh router into MikroTik port 1. Two or three minutes later
   everything was back.

```routeros
/ip address add address=10.20.0.1/22 interface=bridge disabled=yes comment=lan-final
/ip dhcp-server add name=lan-dhcp interface=bridge address-pool=lan-pool disabled=yes
/system script add name=cutover source=":if ([/ip address get [find comment=\"lan-final\"] disabled]) do={ /ip address set [find comment=\"lan-final\"] disabled=no; /ip dhcp-server set [find name=\"lan-dhcp\"] disabled=no; /log warning \"CUTOVER done\" }"
/ip dhcp-client set [find interface=ether1] script=":if (\$bound=1) do={ /system script run cutover }"
```

## Checked afterwards

- Desktop and server kept their addresses, gateway is now the MikroTik
- Pi-hole still resolving and blocking for the house
- Cloudflare tunnel and Tailscale both up, media server reachable from outside
- Router log shows the cutover script fired a second after the WAN lease

## Things I learned

- **My ISP uses carrier-grade NAT.** The WAN address is in the 100.64.0.0/10
  range, so port forwarding was never an option. Good thing everything I
  expose goes out through a tunnel instead.
- **Keep a back door.** The factory management address stays on the router as
  a fallback, and the router's port 5 stays empty for plugging a laptop in.
- **Keys, not passwords.** Management from the server uses an SSH key now. The
  sticker password went into my password manager.
- **Bridge mode costs you the mesh app's router features.** Device list,
  pausing devices and its guest network all stop. That's the router's job now.

## Next

Phase 3: put the server on its own VLAN and block the IoT devices (pet camera,
doorbell chimes, streaming sticks) from reaching it at the firewall. Separate
IoT and guest Wi-Fi comes later with a VLAN-capable access point, since the
mesh router can't tag Wi-Fi networks.
