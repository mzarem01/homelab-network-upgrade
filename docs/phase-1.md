# Phase 1: physical install (2026-10-18)

No configuration in this phase. The old mesh router stays the router. The goal
was to get everything on the UPS, swap the old unmanaged switch for the Zyxel,
and clean up a corner that had power strips plugged into power strips.

## Before

- Mesh router doing everything (gateway, DHCP, Wi-Fi), no VLAN support
- A 5-port unmanaged switch upstairs linking the router, the server, a wall
  run, and a run down to the basement where my workstation lives
- Daisy-chained power strips behind the TV stand. That one was a fire risk.

## UPS outlet plan

The UPS has 5 battery outlets and 4 surge-only. Battery goes to the things that
are still useful during an outage. The TV dies anyway, so it doesn't get one.

| Battery + surge | Surge only |
|---|---|
| Server | Backup USB drive (only busy at 5am) |
| Media USB drive (part of the server's storage pool) | TV, streaming stick, lamp |
| MikroTik router | |
| Short extension cord: fiber box, mesh router, switch | |

Two things I didn't expect:

1. **Wall-wart bricks block the outlets next to them.** I could only fit about
   four plugs on the battery side. A short plain extension cord (no surge
   protection, so it's allowed) carries the small bricks. A pack of 1 ft outlet
   extension cords is the cleaner fix.
2. **The switch had to go on battery for now.** In this phase the switch is the
   only thing connecting the mesh router to the server. If it loses power, the
   server stays up but nobody can reach it, and since the server runs Pi-hole,
   DNS dies for the whole house. Once the MikroTik is the router, the server
   plugs into it directly and the switch can drop back to surge-only.

Never plug the UPS into a power strip, and never plug a power strip into the
battery side. CyberPower says no to both.

## Switch ports (Phase 1)

| Port | Goes to |
|---|---|
| 1 | Mesh router |
| 2 | Run to the basement switch (workstation) |
| 3 | Wall run |
| 4 | Server |
| 5-8 | Spare |

## What went wrong

**The house lost "internet" while the server was off.** I shut the server down
cleanly to move it. Every device in the house uses Pi-hole on the server for
DNS, so with the server off, nothing could resolve names. The internet was fine,
DNS wasn't. Good to know before an outage teaches me.

**Port 2 stayed dark.** The basement cable looked plugged in. It wasn't clicked
in. Reseated, link came up.

**I pulled power on a mounted drive.** I swapped the two USB drives' power bricks
while the server was running. The media drive dropped off, came back under a new
device name, and the old mount was left pointing at a device that no longer
existed. The storage pool went down with it. Recovery:

1. Stop every container that uses the pool so nothing writes into an empty folder
2. Lazy-unmount the stale mount
3. `e2fsck -p` on the drive (journal replayed, clean)
4. Mount it again by UUID
5. Restart the mergerfs pool through systemd (see lessons for why)
6. Start the containers and check each one sees the files

No data lost. The rule now: unmount first, then unplug.

**The backup drive wouldn't come up.** Its built-in USB hub showed up but the
disk behind it kept failing with "Cannot enable. Maybe the USB cable is bad?"
Swapping power bricks proved power wasn't it. A full reseat of the cable at both
ends plus a power cycle fixed it. My backup script has a mount guard, so if the
drive had stayed missing, the nightly job would have aborted instead of filling
the server's main disk.

## After

- UPS online, about 10% load, roughly 100 minutes of runtime
- Server shuts itself down on low battery (see [ups-nut.md](ups-nut.md))
- BIOS set to power back on when AC returns, confirmed: the server booted the
  moment I switched the UPS on
- Old switch retired, both USB drives mounted, all services back
