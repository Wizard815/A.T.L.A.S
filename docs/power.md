# Power

A battery of its own, charged by the truck while it runs, so the starter battery
is never used. Battery choice is still open: PiJuice HAT (one board, Li-ion, keep
out of the sun) or Witty Pi + small 12 V LiFePO4 (6 to 10 Ah) + small DC-DC
charger (better in heat).

| Mode | When | Draw |
| --- | --- | --- |
| Parked | ignition off | Pi off. Only while something is unsent: the ESP32-C3 scans for about 2 s an hour (about 0.02 Wh a day) |
| Driving | ignition on | about 1.5 to 3 W |
| Share | turned on from the app or the button | about 2 to 4 W, until off, a timeout, or 12.4 V |

What goes wrong:

- Without a clean shutdown the boot disk corrupts. The Pi always shuts down
  itself; power is only cut by the HAT after it halts.
- Boot from the USB SSD, not the SD card (on a 3B: `program_usb_boot_mode=1` once).
- A cab in summer passes 60 °C. Mount the box under a seat.
- Every 12 V run gets a fuse at the battery end.
