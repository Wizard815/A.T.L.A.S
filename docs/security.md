# Theft and find my truck

## Find my truck

- Every sync sends the last GPS fix, so HA and Overseer always have the last
  known place, even when the truck is off.
- DASH app and device: "Find truck" shows the last fix and its age. If the Pi is
  on, it also asks for a live fix.
- HA can also show it as a device tracker; Traccar or OwnTracks work too.

## Detecting theft

| Sign | How |
| --- | --- |
| Ignition on, no owner nearby | no paired phone (Bluetooth), DASH device or fob start in the last 2 minutes |
| Moving with the ignition off (towed) | GPS moves more than 100 m, or the fob module's accelerometer sees tilt |
| Box opened or unplugged | reed switch on the box lid; loss of truck power while parked |

When it triggers, truckd sends an alert over LTE (phone, HA, Overseer) and sends a
GPS fix every 30 s until you clear it from the app. With no LTE signal it sends
over Meshtastic.

Remote start never counts as "owner nearby" by itself, since anyone could drive
off after a start (the truck still needs the key to shift out of Park).

## Waking the Pi over LTE

The modem is on the Pi, so while the Pi is off there is no LTE. Two ways to fix
that:

1. **Modem stays in standby (recommended).** Keep the EC25-AF powered in sleep
   mode while parked (about 10 to 25 mA from the reserve, about 7 Wh over 72 h).
   A text message to its number pulses its RI (ring) pin, which wakes the Pi
   through the Witty Pi. truckd reads the text and acts only if it's signed by
   your phone (`find`, `start`, `lock`).
   - Turn it on per park: "away mode" in the app, or always for the first 24 h.
   - Check your hotspot line can receive texts. Some data-only lines can't.
2. **A second, small cellular modem on the fob module** (LTE-M, for example
   SIM7080G, about $15 plus an IoT SIM at $2 to $5 a month). Lower power, but a
   second plan.

The fob module itself doesn't need LTE for its schedule. A text only matters for
starts or a find while you're away from the truck.
