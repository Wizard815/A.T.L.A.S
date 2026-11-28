# Architecture

Everything in the truck talks only to the Pi, so a module never needs to know
whether home is reachable. The Pi picks the best link it has and sends as much
as that link can carry.

## The brain

| Part | Does | Connects to the Pi |
| --- | --- | --- |
| Witty Pi 4 Mini | clock, timed wake, power off, wake on ignition | GPIO header |
| Built-in Wi-Fi | truck network and `Truck-Share`, channel 6, up only while driving or in Share mode | built in |
| USB Wi-Fi adapter + antenna | finds home or the phone hotspot | USB |
| USB SSD 1 to 2 TB | NAS, log, boot disk | USB |
| LTE modem HAT (Quectel EC25-AF mPCIe) | its own data link for sync and a truck hotspot; its built-in GNSS replaces the USB GPS | USB (cable, not pogo pins) + 5 V straight from the buck |
| Weather: BME280 inside the box + DS18B20 probe outside | pressure and humidity; outside temperature on a long cable | I2C (GPIO 2/3, shared with the Witty Pi, no address clash) + 1-Wire (GPIO 17) |
| Battery monitor | INA226: voltage and current of the LiFePO4 | I2C |
| Ignition sense | PC817 from ACC | Witty Pi wake input + a GPIO |
| ESP32-C3 home scanner / DASH bridge | scans while the Pi is off; ESP-NOW bridge while it's on | UART (`dtoverlay=miniuart-bt`, `/dev/ttyAMA0`) + Witty Pi wake input |
| Meshtastic RAK4631 | mesh messages, always on at about 10 mA | USB serial |
| WiCAN | OBD2 | truck Wi-Fi, MQTT |
| Fob module (own mod, own battery) | remote start, lock, unlock, schedule | ESP-NOW through the bridge |

## Where everything plugs in

The Pi 3B has four USB ports. The X852 takes one and gives back two through its own USB hub, so there are spare ports:

| Port or pins | What |
| --- | --- |
| USB 1 | Geekworm X852 dual mSATA board (one USB 3.0 link with a built-in hub, plus 2 extra USB ports). Slot 1: 64 GB mSATA, the NAS and boot disk. Slot 2: the second 64 GB for camera clips, or keep it as a spare. truckd deletes synced trip files oldest first when space drops below 20 % |
| X852 USB ports | spare: a USB GPS or Wi-Fi adapter later |
| USB 2 | Wi-Fi adapter with antenna (finds home) |
| USB 3 | LTE modem HAT (GNSS too, read by gpsd from its NMEA port) |
| USB 4 | Meshtastic radio |
| GPIO under the Witty Pi | ESP32-C3 (UART + wake line), weather sensors, ignition sense: a stacking header or short wires to the pins the Witty Pi doesn't use |
| Wireless | WiCAN (OBD2) and DASH join the truck Wi-Fi; overland modules later do the same |

Services on the Pi: `truckd` (Go), Mosquitto, gpsd, ModemManager + NetworkManager (LTE), copyparty (Truck Share), ffmpeg for video conversion.

## Powering the X852

The X852's 5 V DC jack (5.1 V, 4 A) can power the Pi through a 2-pin cable to the
GPIO 5 V pins. Don't do that with the Witty Pi: it would feed the Pi around the
Witty Pi's switch, so the Pi could never turn off.

- Feed the Witty Pi from the 5 V buck (4 A or more). It switches the Pi's 5 V.
- Power the X852 from the Pi's GPIO 5 V with its 2-pin cable (power flows
  Pi to X852), so the SSDs turn off with the Pi.
- Never power the Pi from micro USB as well. Geekworm says micro USB (2.5 A
  max) may not give the SSDs enough current.
- Check the stack fits: X852 under the Pi on standoffs, Witty Pi 4 Mini on the
  header, LTE HAT beside it on a cable.

## LTE modem and truck hotspot

A Quectel EC25-AF (the US variant) in an mPCIe-to-USB HAT gives Atlas its own
data link and lets the truck Wi-Fi share it.

- truckd picks links in order: home Wi-Fi, LTE, phone, Meshtastic. LTE use is
  counted against a monthly cap, and only small data goes over it: see
  [lte-data.md](lte-data.md).
- Hotspot: while the Pi is on, the owner can turn on internet for the truck
  Wi-Fi from the app (`POST /api/hotspot`). Guests on `Truck-Share` get files
  only unless the owner allows internet for them too.
- The modem is off while the Pi is off (switched with the Pi's 5 V), so it costs
  nothing parked.
- Use a data SIM that allows hotspot or modem use (many phone plans forbid it).

What goes wrong:

- The modem pulls about 2 A in short peaks. Feed the HAT from the 5 V buck
  (3 A or more), not through the Pi, or the Pi browns out and reboots.
- The pictured HAT has one SMA socket. Use a board with two (main + diversity)
  and a 2-in-1 LTE roof antenna, or signal at the edge is much worse.
- Check the HAT, the Witty Pi and the X852 fit together. The simplest layout is
  the modem HAT beside the Pi on a USB cable, not stacked.
- No signal means no LTE. In the woods, Meshtastic and satellite still carry the
  heartbeat.

## Links home

| Link | Sends |
| --- | --- |
| Home Wi-Fi + WireGuard | everything, including files |
| LTE modem, data SIM (metered, monthly cap in config) | events, state, trip summary, small photos; big files wait for home |
| Phone hotspot or Bluetooth tether (metered) | same, if the modem has no signal or no SIM |
| Meshtastic, APRS, Iridium | heartbeat and alarms |
