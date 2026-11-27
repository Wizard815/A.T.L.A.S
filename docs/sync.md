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
   3. files to Unraid, `trips/<date>-<name>/` (skipped on metered links);
   4. pull known networks, module config and firmware.
6. **Mark done** per source, then shut down if the ignition is off and Share mode is off.

Known networks are tagged `home`, `phone` or `other`, plus `metered`.
