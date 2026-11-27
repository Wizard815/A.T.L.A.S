# A.T.L.A.S: Adventure Tech: Link, Archive, Sync

A small homelab that lives in the truck. A Raspberry Pi 3B sleeps fully off while
parked, wakes to look for home, and syncs the trip to the home lab: records into
Overseer, files to Unraid, live state into Home Assistant. While driving or in
Share mode it runs the truck's own Wi-Fi, a guest photo and video share, GPS,
weather and OBD2.

Design plan: https://claude.ai/artifact/9NbqhnBu763WGw4LTWEkMh

## Parts of the system

| Part | What | Status |
| --- | --- | --- |
| Brain | Pi 3B + Witty Pi 4 Mini power HAT, USB SSD, two Wi-Fi radios | planned |
| `truckd` | Go service on the Pi: store, sync, API, GPIO, remote start | planned |
| OBD2 | MeatPi WiCAN (bought), or the DIY ESP32 in `modules/obd2` | planned |
| Home scanner | ESP32-C3 that scans for home Wi-Fi while the Pi is off and wakes it; also the ESP-NOW bridge for DASH (firmware lives in D.A.S.H) | planned |
| Meshtastic | RAK4631 on USB, runs on its own | planned |
| Truck Share | guest web page for photos and videos; owner tools in the APK | planned |
| Overland modules | ESP32 modules (power, water, fridge, level, loads, camp watch) | later |

## Layout

```
docs/        architecture, power, sync, protocol, share, parts
protocol/    message envelope schema (C header and Go types generated from it)
brain/
  truckd/    Go service
  os/        Pi setup scripts: Witty Pi schedule, USB boot, Wi-Fi radios, WireGuard
share-web/   guest share page (Svelte)
app/         owner APK (Android)
modules/
  lib/       shared ESP32 library: ring log, MQTT, pairing, OTA
  obd2/      WiCAN config and DIY ESP32 firmware
hardware/    wiring and enclosure notes
home/        Overseer token, Home Assistant package, Unraid share, mesh gateway
```

## Connects to

- **Overseer**: `POST /api/ingest?name=truck`, record ids `truck:<module>:<seq>`.
- **Home Assistant**: MQTT bridge with discovery; car care and travel book package.
- **Unraid**: trip and share files under `trips/<date>-<name>/`.
- **D.A.S.H**: the Atlas mod, `dash/v1/atlas/<type>` (spec in D.A.S.H `docs/mods/truck-protocol.md`).
- **FAITH**: remote start needs a Yes.

## License

MIT, see [LICENSE](LICENSE).
