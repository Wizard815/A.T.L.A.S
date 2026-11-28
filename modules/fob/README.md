# Fob module

An Atlas mod: an ESP32 with its own battery that presses a spare key fob's
buttons on a schedule or on command. See [docs/remote-start.md](../../docs/remote-start.md)
for what it does and the rules it keeps.

## Parts

| Part | For |
| --- | --- |
| ESP32-C3 board with an antenna connector (deep sleep about 10 µA) | the module |
| Spare GM key fob, programmed to the truck | the buttons |
| 3 PC817 optocouplers + 1 kΩ resistors | lock, unlock, remote start buttons |
| LiFePO4 18650 cell, about 1500 mAh, with protection | its own battery |
| LiFePO4 charger board with temperature cutoff (no charging below 0 °C) | charges from the truck |
| 5 V buck from ACC, fused | charger feed, only when the truck is on |
| 3.3 V buck-boost (for example TPS63020) | ESP32 and fob power from the cell |
| DS3231 clock module (I2C) | keeps time with the Pi off |
| PC817 from ACC | ignition sense |
| Small box, mounted away from the main box | install |

## Wiring

- Solder an optocoupler across each fob button; each is driven by a GPIO through
  a 1 kΩ resistor. The fob has its own ground.
- Power the fob from the 3.3 V rail through a diode (about 3.0 V) instead of its
  coin cell.
- ACC sense on a GPIO and the ESP32 wake pin, so it stays awake while driving.

## Power

| State | Draw |
| --- | --- |
| Parked: deep sleep, 10 s check-ins, DS3231 | about 0.2 mA |
| Driving | awake, charging |
| A start | well under 1 mAh |

A 1500 mAh cell runs it for months parked. It charges only while the truck runs.
At 10 % it stops scheduled starts (it says why) and keeps the clock running.

## Firmware

ESP-IDF or Arduino, in `firmware/` here (not written yet). Frames follow the
module protocol (`docs/protocol.md`) with `id: "fob"`; the bridge forwards them
to truckd over the UART.
