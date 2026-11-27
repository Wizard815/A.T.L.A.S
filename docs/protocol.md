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
`obd`, `dtc`, `dtx`, `sync`, `wx`, `pwr`, `gps`, `bye`. The spec lives in D.A.S.H
`docs/mods/truck-protocol.md`.
