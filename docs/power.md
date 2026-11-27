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

What goes wrong:

- The ESP32-C3 scanner must run from an always-on 5 V feed, not the Witty Pi's
  switched output, or it can't scan while the Pi is off.

- Without a clean shutdown the boot disk corrupts. The Pi always shuts down
  itself; power is only cut by the HAT after it halts.
- Boot from the USB SSD, not the SD card (on a 3B: `program_usb_boot_mode=1` once).
- A cab in summer passes 60 °C. Mount the box under a seat.
- Every 12 V run gets a fuse at the battery end.
