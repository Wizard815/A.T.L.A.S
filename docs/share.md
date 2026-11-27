# Truck Share

- Guests scan a Wi-Fi QR to join `Truck-Share`; the share page opens (a second QR opens it if not).
- The page: photo and video grid, swipe full screen, videos play in the phone's own player,
  select several to download, upload from the camera roll. No app needed.
- Guests can't delete or rename. Only the owner key can, from the Atlas mod in the DASH phone app; the Pi checks it.
- Guest Wi-Fi reaches only the share. Remote start, settings and module commands need the owner key.
- At home files sync to Unraid; Home Assistant's Media browser shows them.
- Speed: about 3 to 5 MB/s on a 3B, 20 or more phones.

## Videos phones can't play

Uploads a phone browser can't play (MKV, some HEVC) are converted in the
background to H.264 MP4 with ffmpeg (`h264_v4l2m2m`, the Pi's hardware
encoder). The original is kept; both are served with `Range`.

- The Pi 3B decodes HEVC in software, so a 1080p HEVC clip converts slower than
  real time. Conversions queue and run only while the truck is on or in Share
  mode, never on the battery-only hourly wake.
- Until the MP4 is ready, the viewer shows the poster frame and "converting".

## Viewer and owner mode

Same feel as the DASH share page: a grid, then a swipe-through viewer for photos
and videos that the phone plays itself. A separate owner QR (shown in the DASH
app's Atlas mod) opens the same page in owner mode: delete, hide, Share mode.
