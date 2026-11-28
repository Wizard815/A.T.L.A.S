# Remote start, lock and unlock (fob module)

Remote start is its own Atlas mod, not part of the main box: the **fob module**,
an ESP32 with its own battery and a spare key fob inside. It runs all the time,
keeps the schedule itself, and starts the truck even when the Pi is off. Build
notes are in [modules/fob](../modules/fob/README.md).

The truck's own remote start does the starting, so its safety checks (in Park,
hood closed, shuts off after its run time) stay in charge.

## What it does

| Action | Fob presses |
| --- | --- |
| Start | lock, then hold remote start about 4 s (GM's sequence) |
| Lock | lock once |
| Unlock driver | unlock once |
| Unlock all doors | unlock twice (the truck's default; check "Remote unlock" in the truck's settings) |

## Schedule

The schedule lives on the fob module, so it works with the Pi off, no signal and
no phone.

- Up to 8 entries: days of the week, time, action (start, lock, unlock driver,
  unlock all).
- **Pause:** skip the next one, skip a date (a day off), or pause everything
  until a date (vacation). Set from the app the night before.
- Set and changed from the DASH phone app or the DASH device (Atlas mod) while
  the Pi is on. truckd passes it to the fob module, which confirms and stores it.
- Later: Home Assistant can add skip dates from your calendar (days off,
  holidays) the next time the truck syncs.

## Manual commands

While the Pi is on, the DASH phone app or the DASH device can start, lock or
unlock right away through Atlas. Near the truck with the Pi off, the phone can use
Bluetooth to the bridge, which wakes the Pi first (about 30 to 60 s).

| From | Path | Delay |
| --- | --- | --- |
| DASH phone app on the truck Wi-Fi, or over LTE/home | truckd API, then ESP-NOW to the fob module | up to 10 s |
| DASH device in the cab ("Fob" page) | truckd HTTP API with its own owner token (`DASH_ATLAS_OWNER_TOKEN`, paired like a phone), so a spoofed ESP-NOW frame can never start the truck | up to 10 s |
| Phone near the truck, Pi off | Bluetooth to the bridge, wakes the Pi | 30 to 60 s |
| Schedule | the fob module on its own | none |

Every manual start needs a confirm on the phone or a long press on DASH.

## Time

- The Pi sets its clock from NTP whenever it has LTE or home Wi-Fi (chrony), then
  sets the Witty Pi clock.
- truckd sends the time to the fob module each time they talk, and the fob
  module keeps it on a DS3231 clock chip (the ESP32's own clock drifts minutes a
  day in deep sleep).
- If the fob module hasn't had a time update for 30 days, it still runs the
  schedule (a DS3231 drifts about a minute a year) and shows a warning.

## Rules the fob module enforces

These live on the fob module, since it often acts while the Pi is off.

- **Never start in a closed garage.** At home, a scheduled start asks Home
  Assistant first (see below). With no answer from HA, it doesn't start at home
  unless the home spot is marked outside. Carbon monoxide in an attached garage
  can kill.
- Not when the ignition is on, or the last start was under 10 minutes ago.
- At most 3 starts a day without a fresh confirm.
- Every action is logged with its reason (schedule, app, DASH) and sent to
  truckd, then Overseer and Home Assistant.

## Garage door (later)

When the fob module sees the home Wi-Fi, it joins it before a scheduled start and
asks Home Assistant to open the garage (MQTT `atlas/fob/garage`, HA automation
opens the cover and answers when it's open). Only then does it start the truck.
If the garage doesn't report open within 60 s, it doesn't start and tells you.

## How a command reaches the fob module

- The fob module talks only to the ESP32-C3 bridge over encrypted ESP-NOW
  (channel 6). truckd sees it as `truck/fob/*` (module protocol).
- While parked, it wakes every 10 s and sends a `kind: "poll"` frame to the
  bridge, then listens about 50 ms. The bridge holds up to 4 truckd commands for
  the fob (30 s each) and sends them the moment a poll arrives. With the Pi off the bridge doesn't answer and the fob
  module goes back to sleep.
- Commands carry a nonce and expire after 30 s, and are signed with HMAC-SHA256
  using the fob key (set when the fob module is paired). Old or repeated
  commands are refused.

## Bluetooth from the phone

The bridge (firmware in D.A.S.H `docs/mods/atlas.md`, "Remote start over
Bluetooth") listens while the Pi is off (about 5 mA: it wakes every 6 s and
listens for 1.5 s). Expect about 30 to 100 m to a phone.

- The phone signs `remote_start|<nonce>|<exp>` with the start key it got at
  pairing. The bridge refuses bad signatures, repeated nonces (last 8), more
  than one start a minute, and `exp` more than 120 s ahead or in the past.
- The bridge wakes the Pi and sends the frame after its `hi`. truckd checks it
  and passes a start to the fob module.
- A start the Pi never wakes for is dropped after 5 minutes.
