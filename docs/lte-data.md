# LTE data budget

The modem is a Quectel EC25-AF (Verizon-certified) on a bundled dedicated
hotspot line: 15 GB of high-speed data a month. truckd keeps Atlas well under it.

## What goes over LTE

| Data | Over LTE | Size |
| --- | --- | --- |
| Frames, events, heartbeat, `pwr`, `gps`, `wx` | yes | a few MB a month |
| Logs, GPX tracks, trip summaries | yes, compressed | tens of MB a month |
| OBD2 history | yes, batched every 15 min | small |
| Photo thumbnails (so HA's travel book shows the trip) | yes, if under 70 % | about 50 KB each |
| Full photos and video | no, held for home Wi-Fi | |
| Guest Truck-Share internet | only if the owner turns it on, throttled | see below |
| OS and container updates | no, home Wi-Fi only | |

## Counting

- truckd reads the modem interface's byte counters (`wwan0`) every minute and
  keeps a monthly total on the SSD, so it survives reboots.
- The month resets on the plan's billing day (`lte.billing_day`, default 1).
- `GET /api/status` and the `sync` frame show `lte_used` and `lte_cap` so DASH and
  HA can show them. HA gets a sensor `sensor.atlas_lte_used`.
- The carrier's count can differ slightly from the modem's. The default cap is
  14 GB so there is room.

## Limits

| Used | What truckd does |
| --- | --- |
| under 70 % | everything in the table above |
| 70 % | stop thumbnails; guest internet drops to 1 Mbit/s |
| 80 % (about 12 GB) | warning to the phone, DASH and HA; guest internet off |
| 95 % | only frames, events and heartbeat; owner hotspot still works, with a warning |
| 100 % of the cap | LTE off for sync except alarms (about 1 KB each) until the month resets |

Guest internet, when the owner allows it: each guest device gets 2 Mbit/s and
200 MB a day (`tc` on the Truck-Share interface), and video streaming sites
can't be blocked reliably, so the caps do the work.

## Settings

```yaml
lte:
  apn: vzwinternet
  cap_gb: 14
  billing_day: 1
  warn_percent: 80
  guest:
    allowed: false
    mbit: 2
    mb_per_day: 200
```

What goes wrong:

- A dedicated hotspot line may be tied to the carrier's hotspot device. Ask the
  carrier to move the line to the SIM in the modem (give them the modem's IMEI);
  the EC25-AF is on Verizon's certified list.
- Phones on the truck Wi-Fi do background updates and photo backups. With guest
  internet on, these eat the budget first; that's why the per-device caps exist.
- "Unlimited" plans slow down after their high-speed data; truckd doesn't see
  that, so keep the cap at the plan's high-speed amount.
