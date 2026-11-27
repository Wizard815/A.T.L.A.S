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
| BME280 | temperature, humidity, pressure | I2C, GPIO 2 and 3 |
| Ignition sense | PC817 from ACC | Witty Pi wake input + a GPIO |
| Remote start | optocoupler across the starter input or a spare fob | GPIO |
| ESP32-C3 home scanner / DASH bridge | scans while the Pi is off; ESP-NOW bridge while it's on | UART (`dtoverlay=miniuart-bt`, `/dev/ttyAMA0`) + Witty Pi wake input |
| Meshtastic RAK4631 | mesh messages, always on at about 10 mA | USB serial |
| WiCAN | OBD2 | truck Wi-Fi, MQTT |

Services on the Pi: `truckd` (Go), Mosquitto, gpsd, copyparty (Truck Share), ffmpeg for video conversion.

## Links home

| Link | Sends |
| --- | --- |
| Home Wi-Fi + WireGuard | everything, including files |
| Phone hotspot or Bluetooth tether (metered) | events, state, trip summary |
| Meshtastic, APRS, Iridium | heartbeat and alarms |
