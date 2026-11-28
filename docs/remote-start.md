# Remote start, lock and unlock

A spare key fob for the 2017 Colorado lives inside the box, with its buttons wired
to the Pi through optocouplers. The truck's own remote start does the starting,
so its safety checks (in Park, hood closed, shuts off after its run time) stay in
charge.

## Wiring

- A spare fob programmed to the truck. If it's the factory GM fob (it has the
  curved-arrow remote start button), a locksmith or dealer can program a spare;
  some GM trucks also allow a do-it-yourself procedure with two working keys.
- Open the fob and solder a PC817 optocoupler across each button: lock, unlock,
  remote start. Each optocoupler is driven by a Pi GPIO through a 1 kΩ resistor.
  The fob never shares a ground or voltage with the Pi.
- Power the fob from a 3.0 V regulator instead of its coin cell, or keep the coin
  cell and change it yearly.
- GM's remote start is press lock, then hold remote start for about 4 s.
  `truckd` does that sequence.

## When it starts

| Trigger | How it reaches the truck | Delay |
| --- | --- | --- |
| Scheduled (for example 7:40 on weekdays) | `truckd` sets a Witty Pi wake a minute early, then presses the fob | none |
| DASH phone app, near the truck | Bluetooth to the ESP32-C3, which wakes the Pi | about 30 s (Pi boot) |
| DASH phone app on the truck Wi-Fi | `POST /api/cmd/remote-start` | none if the Pi is on |
| DASH phone app far away | over the internet when the truck has a link, or a Meshtastic message (later) | depends on the link |

Every start from the app needs a confirm on the phone; scheduled starts are set
once in the app.

## Rules truckd enforces

- **Never start in a closed garage.** Scheduled starts are skipped when GPS says
  the truck is at home, unless you've marked the home spot as outside. Carbon
  monoxide in an attached garage can kill.
- Not when the ignition is already on, the reserve battery is under 5 %, or the
  last start was under 10 minutes ago.
- At most 3 starts a day without a fresh confirm.
- Every start, lock and unlock is logged and sent to Overseer and Home Assistant.

## Bluetooth range

Bluetooth comes from the ESP32-C3 (it has BLE), not the Pi, so it listens while
the Pi is off (about 5 mA: it wakes every 6 s and listens for 1.5 s). Use a C3 board with an antenna connector and a small
2.4 GHz antenna on the window. Expect about 30 to 100 m to a phone: the phone's
own small antenna is the limit, so a bigger truck antenna helps only so much. The
truck Wi-Fi reaches further when the Pi is on.

How a Bluetooth start is checked (bridge firmware: D.A.S.H `docs/mods/atlas.md`,
"Remote start over Bluetooth"):

- The phone signs `remote_start|<nonce>|<exp>` with the start key it got at
  pairing. The bridge refuses bad signatures, repeated nonces (last 8), more
  than one start a minute, and `exp` more than 120 s ahead or in the past, so a
  stranger can't wake the Pi.
- The bridge wakes the Pi and sends the frame right after its `hi`. truckd checks
  the signature again, then the safety rules above.
- A start the Pi never wakes for is dropped after 5 minutes.
- The bridge can only judge `exp` once truckd has set its clock (from `ts` in
  truckd's frames), so the Pi must have booted once since the bridge lost power.
