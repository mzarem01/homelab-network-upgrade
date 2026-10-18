# UPS auto-shutdown with NUT

The UPS is useless for the server if nothing tells the server to shut down
before the battery runs out. NUT (Network UPS Tools) does that. The UPS connects
to the server over USB and shows up as a HID device.

## Install (Debian)

```bash
sudo apt install nut
```

## Config

Sample files are in [`../config/nut/`](../config/nut/). The short version:

- `nut.conf`: `MODE=standalone` (the UPS and the thing it protects are the same box)
- `ups.conf`: one UPS called `cyberpower`, driver `usbhid-ups`
- `upsd.conf`: listen on localhost only, nothing on the LAN needs it yet
- `upsd.users`: one `upsmon` user with a random password
- `upsmon.conf`: watch the UPS and run `shutdown -h +0` when it reports low battery

Lock the files down, they hold a password:

```bash
sudo chown root:nut /etc/nut/*.conf /etc/nut/upsd.users
sudo chmod 640 /etc/nut/*.conf /etc/nut/upsd.users
```

## When does it shut down?

When the UPS itself raises its low-battery flag. On this unit that's 10% charge
or 5 minutes of runtime left, whichever comes first. With the current load
that's well over an hour into an outage, which is more than enough to ride out
most blips.

## Checking it

```bash
upsc cyberpower@localhost
```

What I look at:

| Field | Meaning | First reading |
|---|---|---|
| `ups.status` | `OL` on wall power, `OB` on battery, `LB` low battery | OL |
| `battery.charge` | percent | 100 |
| `battery.runtime` | seconds left at current load | 6150 |
| `ups.load` | percent of capacity | 10 |

## Coming back up

The other half of this is the BIOS setting that powers the server back on when
AC returns. Without it, the server shuts down cleanly and then just sits there
off until someone presses the button. I learned that one during a storm.

## Later

- Alert to Discord when the UPS goes on battery (NUT can run a script on events)
- If other machines go on the UPS, switch to netserver mode so they can watch it
  over the network
