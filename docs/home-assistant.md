# Home Assistant

Atlas shows up in Home Assistant as one device, "Atlas truck", through MQTT
discovery. No HACS integration is needed to start.

## How data gets there

- truckd's Mosquitto bridges to the home broker whenever a link home is up (home
  Wi-Fi, or LTE over WireGuard, including the 6 hour light syncs).
- Entities are announced on `homeassistant/<component>/atlas/<object>/config`
  and state goes to `atlas/<object>/state`, all **retained**, so HA keeps showing
  the last values while the truck is away.
- Availability: `atlas/status` is `online` while the bridge is up and `offline`
  (last will) when it drops. The entities stay at their last values and HA
  shows when each last changed.
- OBD2 values come from the WiCAN through truckd, so HA gets clean names and
  units whichever OBD2 reader is fitted.

## Entities

| Entity | From |
| --- | --- |
| `sensor.atlas_speed`, `rpm`, `coolant_temp`, `intake_temp`, `fuel_level`, `odometer` | OBD2 |
| `sensor.atlas_battery_voltage` (truck), `oil_life` (GM mode 22) | OBD2 |
| `sensor.atlas_dtc_count` + attribute `codes` (code and text) | OBD2 |
| `binary_sensor.atlas_check_engine` | OBD2 |
| `sensor.atlas_reserve_battery` (%), `power_source` (truck, overland, reserve) | INA226 |
| `sensor.atlas_outside_temp`, `cab_temp`, `pressure`, `humidity` | weather |
| `device_tracker.atlas` (GPS, with speed and heading attributes) | LTE modem GNSS |
| `sensor.atlas_sync` (synced, partial, waiting), `lte_used` | truckd |
| `sensor.atlas_fob_battery`, `fob_last_action`, `fob_next_start` | fob module |
| `sensor.atlas_cooler_food_temp`, `binary_sensor.atlas_cooler_lid` | cooler |
| `event.atlas_alert` (kind, level, message) | alerts |
| `button.atlas_lock`, `button.atlas_unlock_driver` | fob module, only while the truck is linked |

Car care (oil changes, service reminders), the fuel budget (Firefly) and the
travel book build on these entities with HA automations and the logbook.

## Remote start from HA

Not in the MQTT set. An MQTT button can be pressed by any HA user or automation
and can't be confirmed, so a start stays in the DASH app and device (signed and
confirmed). The one exception is the planned garage flow: the fob module asks HA
to open the garage, which HA does through its own cover, not a start button.

## When to make a HACS integration

Later, and only if HA needs things MQTT can't do safely:

- starting with a confirm (HA's own confirm dialog plus a signed request to
  truckd's API with an owner token);
- services for the fob schedule and pauses (`atlas.skip_next`, `atlas.pause_until`)
  so calendar automations can pause starts on days off;
- a config flow that pairs HA as an owner like a phone.

It would talk to truckd's HTTP API (docs/api.md) through WireGuard and keep the
MQTT entities as they are.

What goes wrong:

- The home broker must allow the truck's bridge login to write only `atlas/#`
  and `homeassistant/+/atlas/#`.
- If the bridge connects without retained messages, HA shows "unknown" until the
  next sync. truckd republishes every entity's state when the bridge comes up.
