# Mods

Atlas mods are separate units that join the truck network (Wi-Fi + MQTT, or
ESP-NOW through the bridge) and speak the [module protocol](protocol.md). The
main box stays the same when you add one.

| Mod | What | Status |
| --- | --- | --- |
| Fob module | remote start, lock, unlock, schedule ([remote-start.md](remote-start.md)) | planned |
| Cooler | food temperatures and lid open | planned |
| Sentry cam | 24/7 driving camera, motion clips when parked | idea |
| Camp cam | IR night camera on a magnet, pan and tilt, alerts to DASH | idea |
| S.C.R.E.E.N | in-dash touchscreen, brother to DASH, voice later | idea |
| Offline maps | your state's maps and trails on the SSD | planned |

OBD2 (WiCAN) and GPS (LTE modem) already cover trip and engine data.

## Cooler

An ESP32-C3 with DS18B20 probes, a reed switch on the lid, and a small battery
or the overland power.

- **1-Wire puts many sensors on one pin.** Every DS18B20 has its own 64-bit
  address, so 2, 3 or 10 probes share one GPIO and one 4.7 kΩ pull-up. Up to
  about 10 probes over a few metres of cable is easy.
- Suggested probes: meat (bottom), drinks or ice (top), and outside the cooler.
- I2C is the same idea for other sensors: many devices on the same 2 pins, each
  with its own address.
- Alerts: food probe above 4 °C for 30 min (food safety), lid open over 2 min,
  battery low. They go to DASH, the phone and HA.
- Waterproof probes on 1 m cables through the lid seal.

## Sentry cam (24/7 dashcam)

A unit of its own with its own battery and storage, so it never drains Atlas.

| Part | For |
| --- | --- |
| Raspberry Pi Zero 2 W | records and detects motion (about $15) |
| Camera Module 3 Wide (front), optional NoIR (cab or rear) | video |
| The spare 64 GB mSATA in a USB mSATA enclosure, or a high-endurance microSD | storage, loop recording |
| PIR sensor | wakes it when parked |
| ESP32-C3 + 12 V LiFePO4 (6 Ah) or a tap on the overland battery | power control, like the bridge |
| 5 V buck | power |

- **Driving:** records all the time in 1 min clips, oldest deleted first. A hard
  brake or bump (OBD2 or an accelerometer) locks the clip.
- **Parked:** the Zero is off. The PIR wakes it through the ESP32-C3, it records
  60 s, keeps the clip, and sends an alert with a snapshot when Atlas is on.
  Running the Zero all day parked costs about 25 to 30 Wh a day, so motion-only is
  what lets it last.
- It joins the truck Wi-Fi when the Pi is on and copies locked and motion clips
  to Truck Share `/private/cam`. Home sync sends them to Unraid with the rest.
- Software: `picamera2` with its built-in motion detection, written to the same
  envelope as other modules.

Easier route: a store-bought dashcam with parking mode and a hardwire kit. You
lose the Atlas alerts and sync, but it works on day one.

## Camp cam (IR, pan and tilt)

The same Pi Zero 2 W design with a NoIR camera, 850 nm IR LEDs, and a pan-tilt
bracket with two servos, in a small box with a magnetic base.

- One 5 m cable carries 12 V (or 5 V); video goes over the truck Wi-Fi, or put
  the camp mast 2.4 GHz antenna up for range.
- Pan and tilt cover all directions you need (two axes); control it from the DASH
  app or the DASH device.
- Detection: motion first; person and animal detection with a small TFLite model
  on the Zero (a few frames a second) later. Alerts go to DASH (buzz and light),
  the phone and the Meshtastic radio if you're away from the truck.
- Night: IR LEDs light about 5 to 10 m. Bigger IR floodlights reach further and
  run from the overland battery.

## S.C.R.E.E.N

An ESP32-S3 touchscreen in the dash (for example a 4.3 or 7 inch ESP32-S3 RGB
display, about $30 to $50), brother to DASH. Name ideas: "Sensors, Controls,
Routes, Engine, Entertainment, Navigation", or "Smart Cab Readout for Every
Expedition Need".

- Shows sensors, OBD2 and codes, fob buttons, cooler, camera alerts, LTE data,
  battery. It uses the same truckd API and owner token as the DASH device, and
  can reuse DASH's `dash_core` library (Atlas, fob and alert parsing, plain C++).
- **Voice:** the ESP32-S3 can be a voice satellite (mic + speaker, wake word on
  the chip). Speech goes over LTE or home Wi-Fi to the Omi and Argos setup at home,
  which has the real models.
- **Offline voice** needs more than a Pi 3B. A used Pi 4 (4 GB, about $35 to $45)
  can run whisper tiny, Piper text to speech and a small model (about 1 to 1.5 B,
  a few words a second). It's a later add-on; the Pi 3B can't do it.

## Offline maps

The 64 GB SSD has room for your whole state.

- OpenStreetMap extracts by state (Geofabrik) are about 0.2 to 1 GB; a vector
  tile file for the state is a few GB, plus contour lines and trail layers.
- truckd serves them from the SSD over Truck Share; OsmAnd or Organic Maps on
  the phone can use them too.
- Updates download at home only (never over LTE).
