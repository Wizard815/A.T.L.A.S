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

`POST /api/pair` `{"name": "Pixel 8"}` → `{"token": "atl_...", "name": "pixel-8"}`.
Shown once; the Pi stores only its hash. Outside the window: 403.

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

`POST /api/cmd/remote-start` (owner) `{"nonce": "<random 16+ chars>", "exp": <unix seconds>}`

- The phone asks for a confirm before sending.
- `exp` at most 30 s ahead; each nonce works once.
- Refused when the ignition is already on or the battery is below 12.4 V.
- Answers `202 {"id": "c_..."}`; the result is published on `dash/v1/atlas/cmd`:
  `{"m":"atlas","t":"cmd","id":"c_...","c":"remote_start","ok":1,"err":""}`.
