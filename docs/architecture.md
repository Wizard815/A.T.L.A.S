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
| u-blox GPS | position, speed, time | USB |
| Weather: BME280 inside the box + DS18B20 probe outside | pressure and humidity; outside temperature on a long cable | I2C (GPIO 2/3, shared with the Witty Pi, no address clash) + 1-Wire (GPIO 17) |
| Battery monitor | INA226: voltage and current of the LiFePO4 | I2C |
| Ignition sense | PC817 from ACC | Witty Pi wake input + a GPIO |
| Remote start | optocoupler across the starter input or a spare fob | GPIO |
| ESP32-C3 home scanner / DASH bridge | scans while the Pi is off; ESP-NOW bridge while it's on | UART (`dtoverlay=miniuart-bt`, `/dev/ttyAMA0`) + Witty Pi wake input |
| Meshtastic RAK4631 | mesh messages, always on at about 10 mA | USB serial |
| WiCAN | OBD2 | truck Wi-Fi, MQTT |

## Where everything plugs in

No extra HAT is needed. The Pi 3B has four USB ports, and they are exactly enough:

| Port or pins | What |
| --- | --- |
| USB 1 | Geekworm X852 mSATA board with a 64 GB mSATA SSD (a second one is the spare): the NAS and the boot disk (OS runs from it). truckd deletes synced trip files oldest first when space drops below 20 %. Check it mounts without taking the GPIO header the Witty Pi uses, and power it from the 5 V supply if it has its own input, since the Pi 3B USB ports share about 1.2 A |
| USB 2 | Wi-Fi adapter with antenna (finds home) |
| USB 3 | GPS |
| USB 4 | Meshtastic radio |
| GPIO under the Witty Pi | ESP32-C3 (UART + wake line), weather sensors, ignition sense, remote start: a stacking header or short wires to the pins the Witty Pi doesn't use |
| Wireless | WiCAN (OBD2) and DASH join the truck Wi-Fi; overland modules later do the same |

Services on the Pi: `truckd` (Go), Mosquitto, gpsd, copyparty (Truck Share), ffmpeg for video conversion.

## Links home

| Link | Sends |
| --- | --- |
| Home Wi-Fi + WireGuard | everything, including files |
| Phone hotspot or Bluetooth tether (metered) | events, state, trip summary |
| Meshtastic, APRS, Iridium | heartbeat and alarms |
