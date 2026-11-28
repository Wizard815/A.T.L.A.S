# Antennas

Each radio needs an antenna made for its band, so one antenna can't serve
everything. But each band can have one swap point and three antennas.

| Radio | Band | Antennas |
| --- | --- | --- |
| Meshtastic | 915 MHz | the main swap system below |
| Wi-Fi (truck network, finding home) and Bluetooth | 2.4 GHz | one antenna per radio, on the back window; a 2.4 GHz omni on the camp mast for Truck Share range |
| LTE (modem HAT) and GPS | 700 to 2700 MHz, 1575 MHz | one 2-in-1 or 3-in-1 roof antenna: two LTE leads (MIMO) and a GNSS lead to the modem; not swapped. A cell booster would help phones and calls but gives Atlas no data, so the modem replaces it |
| Ham or GMRS radio (later) | 144 to 462 MHz | its own roof mount |

Weather is a sensor, not a radio, so it needs no antenna.

## One swap point, three antennas (915 MHz)

- **Roof:** an NMO mount on the roof (waterproof, the standard truck mount), its
  cable running to a bulkhead N-type socket at the box.
- **Daily:** a short 3 dBi whip on the NMO mount.
- **Overlanding while driving:** a taller 5.8 to 8 dBi fiberglass antenna on the
  same NMO mount; it unscrews in seconds.
- **Camp:** unplug the roof cable at the bulkhead and plug in a 10 m LMR-400
  cable (about 1.3 dB lost at 915 MHz) to an antenna on a telescoping mast or
  hoisted up a tree. The 2.4 GHz camp antenna rides the same mast with its own
  cable.

Waterproofing: N-type connectors outside, self-amalgamating tape over every
outdoor joint, and a drip loop where cables enter.

Power limit: a RAK4631 sends at most about 22 dBm, so even an 8 dBi antenna stays
well under the US 915 MHz limit of 36 dBm EIRP.
