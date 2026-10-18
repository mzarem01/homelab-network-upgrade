# Homelab network upgrade

Moving my home network off a consumer mesh router and onto gear I can actually
segment: a MikroTik router, a managed switch, and a UPS that keeps the important
stuff up and shuts the server down cleanly when the battery runs low.

The end goal is VLANs. Right now my server, my workstation, the SIEM VM, the TVs,
the Ring gear and a no-name pet camera all share one flat network. The mesh
router can't do VLANs at all, so that's what this project fixes.

## The gear

| Role | Device | Why |
|---|---|---|
| Router / firewall | MikroTik hEX (RB750Gr3) | Cheap, VLAN-capable, full RouterOS |
| Switch | Zyxel GS1900-8 | 8-port managed, 802.1Q VLANs, no PoE needed |
| UPS | CyberPower 1000VA / 600W AVR | Enough runtime for the network core plus the server |
| Wi-Fi | existing mesh router | Moves to bridge mode, access point only |

## The plan

Three phases. The internet has to work after every one of them, because other
people live here and use it.

| Phase | What | Status |
|---|---|---|
| 1 | Physical install: UPS, switch replaces the old unmanaged one, no config | Done 2026-10-18 |
| 2 | MikroTik becomes the router, same addressing, mesh goes to bridge mode | Next |
| 3 | VLANs: servers, trusted devices, IoT, guests | Later |

Phase 2 keeps the exact same subnet, gateway address and DNS server, so nothing
that depends on them (the server's tunnel, Pi-hole, bookmarks) has to change.

## Docs

- [Phase 1 write-up](docs/phase-1.md): what I did, the UPS outlet plan, and
  what went wrong
- [UPS auto-shutdown with NUT](docs/ups-nut.md): config and how to check it
- [Lessons](docs/lessons.md): the stuff I'd tell myself before starting

## A note on sanitization

All IP addresses and identifiers here are illustrative and consistent with my
[homelab](https://github.com/mzarem01/homelab) repo. No MAC addresses, serials,
passwords or real hostnames. The NUT password in the sample config is a
placeholder.
