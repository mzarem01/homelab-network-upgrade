# Lessons

Things I'd tell myself before starting.

**Plan the UPS outlets before you plug anything in.** Battery outlets are
limited. Put the things that are still useful during an outage on battery:
server, router, switch, Wi-Fi, fiber box. The TV isn't one of them.

**Bricks eat outlets.** Count physical space, not just outlet numbers. Short
outlet extension cords fix it.

**Know what depends on what.** My server runs DNS for the whole house. Turning
it off looks exactly like the internet going down. And in Phase 1 the switch is
the only path between the router and the server, so the switch needs battery
too, even though it's "just a switch."

**Don't plug an unconfigured router into your LAN.** Out of the box the MikroTik
runs its own DHCP server. On a live network that means devices can grab wrong
addresses. Configure it first, connect it second.

**Unmount before you unplug.** Yanking power from a mounted USB drive left the
system holding a mount on a device that no longer existed. Everything above it,
including the storage pool, failed with I/O errors until I cleaned it up.

**mergerfs is a FUSE filesystem, so who starts it matters.** I first remounted
it from inside a temporary privileged container. The mount looked fine, then
was empty, because the FUSE process belonged to that container and died with
it. Restarting the systemd mount unit put the daemon where it belongs.

**Reseat before you replace.** The backup drive threw "maybe the USB cable is
bad" and the obvious move was to buy a cable. A full reseat at both ends plus a
power cycle fixed it. Try the free thing first.

**Ethernet clicks.** If a port light is off, push the plug until it clicks
before troubleshooting anything else.

**A mount guard in the backup script saved me.** It checks the backup drive is
actually mounted before writing. Without it, a missing drive means backups land
on the server's own disk and fill it.
