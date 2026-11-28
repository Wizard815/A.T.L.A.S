# Finding home and syncing

1. **Ignition off:** the Pi stays up on its battery, does a final check (syncs at
   once if home), sends `bye` to DASH, records whether anything is unsent, and
   shuts down.
2. **While unsent:** the ESP32-C3 wakes hourly, scans about 2 s for the known
   networks list. Nothing: back to sleep. A match: it pulses the Witty Pi wake input.
3. **Connect:** the Pi tries up to 10 times (5, 10, 20 s ... capped at 60 s).
4. **Prove home:** WireGuard to Unraid and Overseer `/health`. An SSID alone is
   not trusted.
5. **Sync, small things first:**
   1. records to Overseer, `POST /api/ingest?name=truck`, stable ids so resends are harmless;
   2. live state to Home Assistant over the MQTT bridge with discovery;
   3. files to the copyparty on Unraid over WebDAV (`rclone sync`), `trips/<date>-<name>/` (skipped on metered links);
   4. pull known networks, module config and firmware.
6. **Mark done** per source, then shut down if the ignition is off and Share mode is off.

## Light sync over LTE while parked

Home can be days away, so the truck also checks in over LTE without waiting
for a home network.

- While data is waiting, the ESP32-C3 also wakes the Pi every 6 hours
  (`BRIDGE_LTE_SYNC_EVERY_H`, 0 = off; DASH `docs/mods/atlas.md`).
- truckd brings up LTE, proves home over WireGuard the same way, and sends only
  steps 1 and 2 plus logs, GPX and weather, within the [LTE budget](lte-data.md).
  Photos and video stay queued for home Wi-Fi.
- It then sends `sync` with `st: "partial"` and shuts down. Any `st` other than
  `synced` keeps the bridge's hourly home Wi-Fi scan going, so the big files
  still go up once the truck is home.
- No LTE signal: it gives up after 2 minutes and tries again at the next wake.
- A home Wi-Fi match always wins over LTE.

Known networks are tagged `home`, `phone` or `other`, plus `metered`.
