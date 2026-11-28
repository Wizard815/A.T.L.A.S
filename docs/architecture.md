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
| Remote start | optocoupler across the starter input or a spare fob | GPIO |
| ESP32-C3 home scanner / DASH bridge | scans while the Pi is off; ESP-NOW bridge while it's on | UART (`dtoverlay=miniuart-bt`, `/dev/ttyAMA0`) + Witty Pi wake input |
| Meshtastic RAK4631 | mesh messages, always on at about 10 mA | USB serial |
| WiCAN | OBD2 | truck Wi-Fi, MQTT |

## Where everything plugs in

The Pi 3B has four USB ports. The LTE modem's GNSS stands in for the USB GPS, so they are still exactly enough:

| Port or pins | What |
| --- | --- |
| USB 1 | Geekworm X852 mSATA board with a 64 GB mSATA SSD (a second one is the spare): the NAS and the boot disk (OS runs from it). truckd deletes synced trip files oldest first when space drops below 20 %. Check it mounts without taking the GPIO header the Witty Pi uses, and power it from the 5 V supply if it has its own input, since the Pi 3B USB ports share about 1.2 A |
| USB 2 | Wi-Fi adapter with antenna (finds home) |
| USB 3 | LTE modem HAT (GNSS too, read by gpsd from its NMEA port) |
| USB 4 | Meshtastic radio |
| GPIO under the Witty Pi | ESP32-C3 (UART + wake line), weather sensors, ignition sense, remote start: a stacking header or short wires to the pins the Witty Pi doesn't use |
| Wireless | WiCAN (OBD2) and DASH join the truck Wi-Fi; overland modules later do the same |

Services on the Pi: `truckd` (Go), Mosquitto, gpsd, ModemManager + NetworkManager (LTE), copyparty (Truck Share), ffmpeg for video conversion.

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
