# truckd HTTP API

For the Atlas mod in the DASH phone app. Status stays on
MQTT (`dash/v1/atlas/*`); pairing, Share mode and commands use this API; files are
served by copyparty.

Base: `http://atlas.local:8080` on the truck Wi-Fi (Pi at `10.42.0.1`), or the
Pi's WireGuard address when away. Errors are `{"error": "what happened and what to do"}`.

## Who can call what

| Caller | Header | Can |
| --- | --- | --- |
| Guest on `Truck-Share` | none | nothing here; guests use copyparty |
| Owner (paired phone) | `Authorization: Bearer atl_...` | everything below |

Guests reach only copyparty; every truckd route returns 403 without an owner key.

## Pairing

Press the box's button: a 2 minute pairing window opens (status LED blinks).

`POST /api/pair` `{"name": "Pixel 8"}` → `{"token": "atl_...", "name": "pixel-8", "start_key": "<64 hex>"}`.
The token is shown once and the Pi stores only its hash. `start_key` is the shared
Bluetooth remote-start key (the same `BRIDGE_START_KEY` the ESP32-C3 holds); the
app keeps it in the phone's keystore. Outside the window: 403. Removing every
owner or `POST /api/start-key/rotate` makes a new key, which truckd sends to the
bridge over the UART only (`{"m":"bridge","t":"key","k":"<64 hex>"}`; the bridge
stores it in flash and answers `{"m":"bridge","t":"key","ok":1}`, or `ok` 0
with `err`).

`GET /api/owners` lists paired phones; `DELETE /api/owners/{name}` removes one.

## Status

`GET /api/status` → the latest `hi`, `obd`, `dtc`, `sync`, `wx`, `pwr`, `gps`
frames in one object, for when MQTT isn't reachable (over WireGuard).

## Truck Share

Files are served by copyparty, not truckd: listing, downloads with `Range`,
thumbnails and resumable uploads all use copyparty's own HTTP API and web page
(see [share.md](share.md)). The DASH app's Atlas mod logs in to copyparty with
the owner account for delete, move and hide.

truckd keeps only the Share controls:

| Route | Who | Does |
| --- | --- | --- |
| `POST /api/share/mode` `{"on": true, "minutes": 120}` | owner | Share mode on or off; `minutes` 0 = until turned off |
| `POST /api/hotspot` `{"on": true, "guests": false}` | owner | share LTE internet on the truck Wi-Fi; `guests` lets `Truck-Share` use it too. Status shows data used this month |
| `GET /api/share/qr` | owner | `{"wifi": "WIFI:T:WPA;S:Truck-Share;P:...;;", "page": "http://atlas.local:3923/", "owner": "<owner login link>"}` |

## Commands

`POST /api/cmd/fob` (owner) `{"c": "start", "nonce": "<random 16+ chars>", "exp": <unix seconds>}`

- `c` is `start`, `lock`, `unlock_driver` or `unlock_all`.
- The phone asks for a confirm before a start.
- `exp` at most 30 s ahead; each nonce works once.
- truckd signs it with the fob key and sends it to the fob module, which
  checks its own rules (see docs/remote-start.md).
- Over Bluetooth the bridge passes a signed frame on the UART instead:
  `{"m":"bridge","t":"cmd","c":"remote_start","nonce","exp","sig","via":"ble"}`.
  `sig` is hex HMAC-SHA256 of `remote_start|<nonce>|<exp>` with the start key.
  For `via: ble`, `exp` may be up to 120 s ahead, since the Pi takes 30 to 60 s
  to boot. The same nonce, rate and safety rules apply.
- Answers `202 {"id": "c_..."}`; the result is published on `dash/v1/atlas/cmd`:
  `{"m":"atlas","t":"cmd","id":"c_...","c":"start","ok":1,"err":""}`.

## Fob schedule

| Route | Does |
| --- | --- |
| `GET /api/fob` | battery, last action, schedule, pauses, clock age (as last reported) |
| `PUT /api/fob/schedule` `[{"days": "mon-fri", "at": "07:40", "c": "start"}]` | up to 8 entries; the fob module confirms before it answers 200 |
| `POST /api/fob/pause` `{"skip_next": true}`, `{"skip": "2026-12-04"}` or `{"until": "2026-12-28"}` | pause; `{"clear": true}` removes pauses |
