# Power

A battery of its own, charged by the truck while it runs, so the starter battery
is never used: a small 12 V LiFePO4 (6 to 10 Ah) charged by a small DC-DC
charger with ignition sense, feeding the Witty Pi. LiFePO4 copes with a hot cab
far better than the Li-ion cells on a PiJuice-style HAT.

| Mode | When | Draw |
| --- | --- | --- |
| Parked | ignition off | Pi off. Only while something is unsent: the ESP32-C3 scans for about 2 s an hour (about 0.02 Wh a day) |
| Driving | ignition on | about 1.5 to 3 W |
| Share | turned on from the app or the button | about 2 to 4 W, until off, a timeout, or 12.4 V |

## The battery is the UPS

The Pi always runs from the small LiFePO4 through the 5 V converter. The truck
only charges that battery (DC-DC charger on ignition). So there is no switch-over
when the ignition goes off: the Pi keeps running on the battery, finishes its last
check and shuts down cleanly.

## Low-battery shutdown (about 5 %)

An INA226 on I2C (about $5) reads the battery's voltage and current, so `truckd`
knows the charge level (LiFePO4 voltage is too flat to judge by eye).

- At about 5 % (around 12.0 V at rest), held for 30 s: `truckd` saves its store,
  stops any sync where it can resume, queues a "low battery" event, sends `bye`
  to DASH, and shuts down.
- It tells the ESP32-C3 not to wake the Pi, and sets no Witty Pi wake, until the
  battery is charging again (above about 13.2 V).
- The ignition coming on always wakes it, since the truck is then charging.

## 72 hours parked in winter, no charging

Target: last 72 hours or more in a cold cab with no charge, doing one sync at the
start and hourly checks after.

| Load | Over 72 h |
| --- | --- |
| One sync at the start, Pi on up to 30 min at 3 W | 1.5 Wh |
| Meshtastic radio, always on | about 4 Wh |
| DC-DC converter idle, Witty Pi and ESP32-C3 asleep, hourly scans | about 4 Wh |
| **Total** | **about 10 Wh** |

A 12 V 6 Ah LiFePO4 holds about 75 Wh. Cold cuts usable capacity by up to half
at -20 °C, so that still leaves about 3 times the need. 6 Ah is enough; pick
10 Ah if you want room for Share mode at camp.

- LiFePO4 must not be **charged** below 0 °C. Buy one whose BMS has
  low-temperature charge protection (or a self-heating one), or set the DC-DC
  charger's low-temperature cutoff. Discharging in the cold is fine.

What goes wrong:

- The ESP32-C3 scanner must run from an always-on 5 V feed, not the Witty Pi's
  switched output, or it can't scan while the Pi is off.

- Without a clean shutdown the boot disk corrupts. The Pi always shuts down
  itself; power is only cut by the HAT after it halts.
- Boot from the USB SSD, not the SD card (on a 3B: `program_usb_boot_mode=1` once).
- A cab in summer passes 60 °C. Mount the box under a seat.
- Every 12 V run gets a fuse at the battery end.
