# OBD2

Default: MeatPi WiCAN, set to MQTT to the Pi's broker, sleep below 13.0 V, wake at 13.5 V.

DIY (CAN trucks, US 2008 and later), read only:

| OBD2 pin | Signal | Goes to |
| --- | --- | --- |
| 16 | +12 V always on | fuse + TVS, automotive buck to 5 V |
| 4, 5 | grounds | GND |
| 6 | CAN-H | SN65HVD230 CANH |
| 14 | CAN-L | SN65HVD230 CANL |

SN65HVD230 to ESP32-S3: TXD to GPIO 5, RXD to GPIO 4, VCC to 3V3, Rs to GND. TWAI at 500 kbit/s.
