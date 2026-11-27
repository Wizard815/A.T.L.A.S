# Truck Share (copyparty)

Truck Share runs on [copyparty](https://github.com/9001/copyparty), the same
file server already running on Unraid (`:3923`). It's one Python 3 file with no
required dependencies, so it runs on the Pi 3B. Pillow and ffmpeg add thumbnails
and video poster frames.

- Guests scan a Wi-Fi QR to join `Truck-Share`, then the page QR (copyparty prints
  one with `--qr`). No app needed.
- copyparty's web page: browse, gallery, view photos, play video in the phone's
  own player (downloads support `Range`), pick several files and download them,
  and resumable uploads from the camera roll.
- Speed: about 3 to 5 MB/s on a 3B, 20 or more phones.

## Accounts and permissions

| Volume | Guests (no login) | Owner account |
| --- | --- | --- |
| `/` (everything shared) | read, download | read, write, move, delete, admin |
| `/drop` (guest uploads) | read, upload | read, write, move, delete, admin |
| `/private` (hidden) | nothing | read, write, move, delete, admin |
| `/dash` (from the DASH handheld) | read, download | read, write, move, delete, admin |

- Guests can't delete or rename: they never get copyparty's `m` or `d` permission,
  and "unpost" (undo your own upload) is turned off.
- Hiding a folder = the owner moves it into `/private`.
- The owner logs in with the owner account. The Atlas mod in the DASH phone app
  keeps that login and opens copyparty in owner mode; an owner QR does the same
  on any phone.
- Guest Wi-Fi reaches only copyparty. truckd's API (remote start, settings,
  Share mode) needs the owner key.

Draft config: [`brain/copyparty.conf`](../brain/copyparty.conf). Passwords live in
`/etc/atlas/copyparty-accounts.conf` on the Pi, never in the repo.

## Files from DASH

DASH keeps a simple share on its own SD card and copies its files here, where
Atlas shows the full version. A `dash` account may only write to `/dash`:

- DASH sends `PUT /dash/<url-encoded name>` to `http://atlas.local:3923` with a
  `PW: <dash password>` header and `Content-Length`, one file at a time, and
  treats any 2xx as done. copyparty identifies the account by its password.
- DASH never deletes anything on Atlas, and the `dash` account can't list or delete.
- A file with the same name is saved under a new name rather than overwritten,
  so DASH keeps its own list of files already sent.
- Test both the PUT and the `PW` header against the copyparty version installed
  on the Pi before relying on them. The DASH side is in D.A.S.H `docs/files.md`
  ("Sync to the Truck Share").

## Videos phones can't play

copyparty doesn't convert video for playback. truckd watches the share and
converts uploads a phone can't play (MKV, some HEVC) to H.264 MP4 with ffmpeg
(`h264_v4l2m2m`, the Pi's hardware encoder), saved next to the original.

- The Pi 3B decodes HEVC in software, so a 1080p HEVC clip converts slower than
  real time. Conversions queue and run only while the truck is on or in Share
  mode, never on the battery-only hourly wake.

## Home side

At home, trip and share files go to the copyparty on Unraid over WebDAV
(`rclone sync`, through WireGuard), into `trips/<date>-<name>/`. Home Assistant's
Media browser and Immich can read the same folder.
