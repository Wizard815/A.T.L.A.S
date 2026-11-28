# Module protocol

Modules talk to the Pi with MQTT over the truck Wi-Fi (Mosquitto on the Pi).
While the Pi is off they keep logging and send the gap when it's back.

| Topic | Direction | Holds |
| --- | --- | --- |
| `truck/<module>/hello` | module to Pi, retained | id, kind, firmware, capabilities |
| `truck/<module>/state` | module to Pi | readings every 10 to 60 s |
| `truck/<module>/event` | module to Pi | alarms and changes |
| `truck/<module>/backlog` | module to Pi | the gap since the last ack |
| `truck/<module>/ack` | Pi to module | highest seq stored |
| `truck/<module>/cmd` | Pi to module | a live command with an expiry |
| `homeassistant/...` | Pi to home broker | HA discovery |

Envelope (`protocol/envelope.schema.json`):

```json
{"v": 1, "id": "fridge", "seq": 4812, "ts": "2026-11-26T21:30:00Z", "kind": "state", "data": {"temp_c": 3.1}}
```

- `id` + `seq` become the Overseer record id (`truck:<id>:<seq>`).
- Commands are never queued, so a stale one can't fire later.
- Each module has its own MQTT login, set when paired.

## DASH (Atlas mod)

`truckd` also publishes dash-mod v1 frames on `dash/v1/atlas/<type>`: `hi`,
`obd`, `dtc`, `dtx`, `sync`, `wx`, `pwr`, `gps`, `bye`. The spec lives in D.A.S.H `docs/mods/atlas-protocol.md`
(Pi wiring and setup in `docs/mods/atlas.md`, added in
https://github.com/Wizard815/D.A.S.H/pull/2). Any frame may carry `ts` (unix
seconds), which DASH uses as its clock on the truck network. The ESP32-C3
bridge firmware lives in D.A.S.H `bridge/`; `truckd` tells it whether a sync is
needed with `{"m":"bridge","t":"need","sync":0|1}` on the UART.

## Alerts

truckd sends alerts to DASH (and S.C.R.E.E.N) as a dash-mod frame on
`dash/v1/atlas/alert` (spec: D.A.S.H `docs/mods/atlas-protocol.md`, "Alerts on DASH"):

```json
{"m":"atlas","t":"alert","id":"a_17","k":"theft","lvl":3,"msg":"Ignition on, no owner nearby","snap":0}
```

- `k`: `cooler`, `cam`, `theft`, `lowbatt`. `lvl`: 1 info (blue), 2 warning
  (amber, repeats every 5 min), 3 urgent (red, repeats every 30 s) until seen.
  `lvl` 0 with the same `id` clears it.
- Keep `msg` under 70 characters so the frame stays under 240 bytes.
- `snap: 1` means a camera snapshot is waiting in the phone app.
- The same alert also goes to the phone, HA and Overseer.
