# truckd HTTP API

For the Atlas mod in the DASH phone app and the guest share page. Status stays on
MQTT (`dash/v1/atlas/*`); files, pairing, share management, uploads and commands
use this API.

Base: `http://atlas.local:8080` on the truck Wi-Fi (Pi at `10.42.0.1`), or the
Pi's WireGuard address when away. Errors are `{"error": "what happened and what to do"}`.

## Who can call what

| Caller | Header | Can |
| --- | --- | --- |
| Guest on `Truck-Share` | none | list and download shared files, upload into `drop/` |
| Owner (paired phone) | `Authorization: Bearer atl_...` | everything below |

Guest requests reach only `/share/*` and the guest upload routes; every other
route returns 403 without an owner key.

## Pairing

Press the box's button: a 2 minute pairing window opens (status LED blinks).

`POST /api/pair` `{"name": "Pixel 8"}` → `{"token": "atl_...", "name": "pixel-8"}`.
Shown once; the Pi stores only its hash. Outside the window: 403.

`GET /api/owners` lists paired phones; `DELETE /api/owners/{name}` removes one.

## Status

`GET /api/status` → the latest `hi`, `obd`, `dtc`, `sync`, `wx`, `pwr`, `gps`
frames in one object, for when MQTT isn't reachable (over WireGuard).

## Truck Share

| Route | Who | Does |
| --- | --- | --- |
| `GET /api/share/folders` | guest, owner | folders, with counts; hidden ones only for the owner |
| `GET /api/share/files?folder=&after=&limit=` | guest, owner | `{items: [{id, name, type, size, taken, folder, url, thumb}], next}` newest first, limit up to 200 |
| `GET /share/f/{id}` | guest, owner | the file; supports `Range` so phones stream video |
| `GET /share/t/{id}` | guest, owner | thumbnail (JPEG, 320 px) or video poster frame |
| `DELETE /api/share/files/{id}` | owner | delete a file |
| `PATCH /api/share/folders/{name}` `{"hidden": true}` | owner | hide or show a folder for guests |
| `POST /api/share/mode` `{"on": true, "minutes": 120}` | owner | Share mode on or off; `minutes` 0 = until turned off |
| `GET /api/share/qr` | owner | `{"wifi": "WIFI:T:WPA;S:Truck-Share;P:...;;", "page": "http://atlas.local/"}` |

## Uploads (resumable)

1. `POST /api/uploads` `{"name": "IMG_2041.jpg", "size": 4823112, "folder": "2026-11-27", "taken": "2026-11-27T14:02:00Z"}`
   → `{"id": "u_...", "offset": 0}`. Guests always land in `drop/`.
2. `PATCH /api/uploads/{id}` with header `Upload-Offset: <n>` and up to 8 MB of
   body → `{"offset": <new>}`. A wrong offset returns 409 with the right one.
3. `HEAD /api/uploads/{id}` → `Upload-Offset` header, to resume after a drop.
4. When `offset == size` the file is checked, thumbnailed and appears in the share.

Unfinished uploads are removed after 24 hours.

## Commands

`POST /api/cmd/remote-start` (owner) `{"nonce": "<random 16+ chars>", "exp": <unix seconds>}`

- The phone asks for a confirm before sending.
- `exp` at most 30 s ahead; each nonce works once.
- Refused when the ignition is already on or the battery is below 12.4 V.
- Answers `202 {"id": "c_..."}`; the result is published on `dash/v1/atlas/cmd`:
  `{"m":"atlas","t":"cmd","id":"c_...","c":"remote_start","ok":1,"err":""}`.
