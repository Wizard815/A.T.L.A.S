# OBD2

Truck: 2017 Chevrolet Colorado. Engine data is on high-speed CAN (pins 6 and 14,
500 kbit/s), which WiCAN reads. GM-only values such as transmission temperature
need GM mode 22 PIDs, added as custom PIDs. GMLAN single-wire CAN (pin 1) is not
needed for engine data.

Default: MeatPi WiCAN ($42, ESP32-C3, Wi-Fi, MQTT, Home Assistant add-on, sleeps
under 1 mA), set to MQTT to the Pi's broker, sleep below 13.0 V, wake at 13.5 V.

DIY (CAN trucks, US 2008 and later), read only:

| OBD2 pin | Signal | Goes to |
| --- | --- | --- |
| 16 | +12 V always on | fuse + TVS, automotive buck to 5 V |
| 4, 5 | grounds | GND |
| 6 | CAN-H | SN65HVD230 CANH |
| 14 | CAN-L | SN65HVD230 CANL |

SN65HVD230 to ESP32-S3: TXD to GPIO 5, RXD to GPIO 4, VCC to 3V3, Rs to GND. TWAI at 500 kbit/s.
