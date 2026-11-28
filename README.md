# A.T.L.A.S: Adventure Tech: Link, Archive, Sync

[![License: PolyForm Shield 1.0.0](https://img.shields.io/badge/license-PolyForm%20Shield%201.0.0-blue)](LICENSE)

A small homelab that lives in the truck. A Raspberry Pi 3B sleeps fully off while
parked, wakes to look for home, and syncs the trip to the home lab: records into
Overseer, files to Unraid, live state into Home Assistant. While driving or in
Share mode it runs the truck's own Wi-Fi, a guest photo and video share, GPS,
weather and OBD2.

Design plan: https://claude.ai/artifact/9NbqhnBu763WGw4LTWEkMh

## Parts of the system

| Part | What | Status |
| --- | --- | --- |
| Brain | Pi 3B + Witty Pi 4 Mini power HAT, mSATA SSD, two Wi-Fi radios, LTE modem HAT | planned |
| `truckd` | Go service on the Pi: store, sync, API, GPIO | planned |
| OBD2 | MeatPi WiCAN (bought), or the DIY ESP32 in `modules/obd2` | planned |
| Home scanner | ESP32-C3 that scans for home Wi-Fi while the Pi is off and wakes it; also the ESP-NOW bridge for DASH (firmware lives in D.A.S.H) | planned |
| Meshtastic | RAK4631 on USB, runs on its own | planned |
| Fob module | ESP32 mod with its own battery and a spare fob: scheduled and manual start, lock, unlock (`modules/fob`) | planned |
| Truck Share | copyparty on the Pi: guests browse, play and upload, can't delete; owner tools in the DASH app's Atlas mod | planned |
| Overland modules | ESP32 modules (power, water, fridge, level, loads, camp watch) | later |

## Layout

```
docs/        architecture, power, sync, protocol, api, share, remote-start, home-assistant, mods, security, antennas, lte-data, parts
protocol/    message envelope schema (C header and Go types generated from it)
brain/
  truckd/    Go service
  os/        Pi setup scripts: Witty Pi schedule, USB boot, Wi-Fi radios, WireGuard
modules/
  lib/       shared ESP32 library: ring log, MQTT, pairing, OTA
  obd2/      WiCAN config and DIY ESP32 firmware
hardware/    wiring and enclosure notes
home/        Overseer token, Home Assistant package, Unraid share, mesh gateway
```

## Connects to

- **Overseer**: `POST /api/ingest?name=truck`, record ids `truck:<module>:<seq>`.
- **Home Assistant**: MQTT bridge with discovery; car care and travel book package.
- **Unraid**: trip and share files to the home copyparty (WebDAV) under `trips/<date>-<name>/`.
- **D.A.S.H**: the Atlas mod, `dash/v1/atlas/<type>` (spec in D.A.S.H `docs/mods/atlas-protocol.md`).
- **DASH phone app**: the Atlas mod is the phone app (owner tools, remote start with a confirm), built against [docs/api.md](docs/api.md). No separate Atlas app.
- **FAITH**: can ask for a Yes on remote start when home.

## License

A.T.L.A.S is source-available under the
[PolyForm Shield License 1.0.0](LICENSE). You can read, run, change and share
it for free, including at work. You can't sell it or offer it as a competing
product or hosted service without permission: open an issue titled
"License request" to ask. [docs/LICENSING.md](docs/LICENSING.md) explains it
in plain English. Versions published before the switch stay under MIT.
