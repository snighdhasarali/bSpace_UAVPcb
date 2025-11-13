# UAV Anomaly Detection PCB — KiCad Project Plan & Deliverables

This document is a complete, practical guide and starter specification you can paste into a KiCad project (v9.x). It contains: BOM, recommended symbols & footprints, schematic net naming, CvPcb mapping table, layout & placement rules for a 2‑layer board, and a checklist for production (Gerbers, DRC, assembly). Use this as the single-source design spec while you build the .sch and .kicad_pcb files.

---

## Project overview

**Objective:** 2‑layer PCB to host a companion board for an onboard inference module (Jetson Nano or Raspberry Pi Compute Module 4), GPS, CSI camera connector, MicroSD holder (optional), power regulators and basic I/O headers.

**Target constraints:**

* Board size: flexible, suggested 80 × 60 mm (adjustable)
* 2 layers (Top = components + signals, Bottom = ground + power pours)
* Keep GPS and camera connectors on board edge for cable access
* Provide a 40‑pin GPIO header footprint (2×20, 2.54 mm) for Jetson Nano / Pi-compatible signals **and** include a CM4/Compute Module camera/PCI footprint option if using CM4

---

## Bill of Materials (BOM) — core components

| Ref         |            Designator | Qty | Comment / footprint suggestion                                                                                                                                    |
| ----------- | --------------------: | --: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| U1          |           GNSS module |   1 | u‑blox NEO‑M8 series SMD module (LCC 12×16 mm) — symbol + 24‑pad footprint (see datasheet)                                                                        |
| U2          |  CSI camera connector |   1 | 22‑pin FFC (Camera Serial Interface) right‑angle or board edge connector (match Pi/CM4 pinout)                                                                    |
| U3          |  SOM / Host connector |   1 | 40‑pin 2×20 header (2.54 mm) for Jetson/PI; optional CM4 22/15‑pin camera connector footprint as required                                                         |
| U4          |     Voltage regulator |   1 | 5V→3.3V regulator — recommended switcher (e.g., MP1584 module footprint or SOT‑23‑5 switching regulator) or SOT‑223 LDO (e.g., AMS1117‑3.3) if simplicity desired |
| J1          |        MicroSD holder |   1 | Push‑push or spring microSD side‑entry footprint                                                                                                                  |
| Cx          |       Decoupling caps |  6+ | 0805 or 0603 close to power pins                                                                                                                                  |
| Lx          |          Ferrite bead |   1 | On 5V input to reduce noise                                                                                                                                       |
| JTAG / UART | Headers / test points |   1 | 2.54 mm header footprints for UART, I2C, SPI, power, and programming                                                                                              |

---

## Schematic sheet structure (suggested sheets)

1. **Power** — battery / 5V input, protection (fuse / reverse diode), bulk caps, regulator (5V→3.3V), 3.3V and 5V nets, test points
2. **SOM Interface** — 40‑pin header symbol (label all pins used: I2C, SPI, UART, GPIO, I2S, SDA/SCL, 5V, 3.3V, GND)
3. **Camera & CSI** — CSI connector symbol + 1.8/3.3V pullups if needed; route MIPI pairs as differential lanes (label lanes)
4. **GNSS** — u‑blox module symbol, UART (TX/RX), PPS, VCC (3.3V), GND, external antenna pad or u.FL connector symbol
5. **Peripherals & Debug** — microSD symbol, UART header, status LEDs, level shifters if needed

---

## Net naming convention (recommended)

Use clear net names and net‑classes in KiCad:

* `GND` — main ground plane
* `3V3` — regulated 3.3V rail (from 5V→3V3 regulator)
* `5V_IN` — external 5V input (battery/buck output if present)
* `USB5V` — if you want separate USB5V
* `GNSS_ANT` — antenna pad/coax
* `CAM_MIPI_L0_P`, `CAM_MIPI_L0_N` etc. — differential pairs for camera lanes
* `SOM_IO_x` — host signals to make mapping obvious

Create net‑classes for: signal (6 mil), I2C/UART (6 mil), power (20 mil), MIPI diff pairs (6 mil with controlled spacing)

---

## Footprint assignment table (for CvPcb)

Below are suggested footprint names you can map in CvPcb. Replace with your preferred library parts if you have them.

* **NEO‑M8 (GNSS module)** — `u‑blox_NEO‑M8_12x16_LCC` (SMD 24‑pad pattern). If using a castellated variant, choose matching castellated footprint.
* **CSI camera connector (22‑pin)** — `FPC_22pin_0.5mm_R/A` (right angle) or `FPC_22pin_0.5mm_top` depending on orientation. Match the Amphenol/official Raspberry Pi camera connector standard.
* **40‑pin GPIO header** — `PinHeader_2x20_P2.54mm` (male right angle or vertical depending whether you need cable or header mating)
* **MicroSD** — `MicroSD_PushPush` or `MicroSD_SidePush` (SMD)
* **Regulator (LDO)** — `SOT223_AMS1117` (if AMS1117), or `SOT23‑5_MP1584` (switcher SOT23‑6 variant — verify exact vendor footprint)
* **Inductors, caps** — `Inductor_0805`, `Cap_0805`, `Resistor_0805`

---

## PCB placement guidelines (2‑layer)

* Put **GNSS** and **CSI** connectors on the board edge to allow antenna/camera cable access. GNSS antenna feed should be away from switching regulator and large digital traces.
* Leave a rectangular keepout for SOM (Jetson/CM4) header and ensure mechanical clearance for a carrier or stacking.
* Place decoupling capacitors within 1–2 mm of every regulator and the GNSS module power pins.
* Place the switching regulator and its inductor on the same side; keep switching loops short.
* Reserve an area on the top for the GNSS antenna (if using passive ceramic antenna on board) and keep digital traces away.
* Route all ground pours on bottom as a solid plane and stitch with multiple VIAs (for 2‑layer use many thermal reliefs and plenty of vias near connectors).

---

## Differential pairs & routing rules

* MIPI CSI lanes: route as differential pairs with matched lengths (±100 ps tolerance) and keep impedance ~100 Ω differential (on 2‑layer this is approximate — keep trace width/spacing small and equal; use the thinnest possible board if controlled impedance is critical). Label lanes `CAM_L0_P/N`, `CAM_L1_P/N`.
* GNSS UART and PPS: single‑ended signals; keep them away from high‑speed switching areas.

---

## Test points & debug

* Add test points for `3V3`, `5V_IN`, `GND`, `UART_TX`, `UART_RX`, `I2C_SDA`, `I2C_SCL`.
* Add 2‑pin header for DFU or boot mode if you expect to program SOM or change boot pins.

---

## Design rule defaults (starting point)

* Minimum track/space: 6 mil (0.15 mm)
* Power traces: 24–30 mil (for up to ~2 A; increase if needed)
* Via: 0.3 mm drill / 0.6 mm finished — use many vias for GND stitching

---

## Gerber / Production Checklist

1. Run ERC and DRC in KiCad
2. Confirm footprints match manufacturer mechanical drawings (especially GNSS and FPC)
3. Export Gerbers (RS‑274X), drill files, and pick‑and‑place
4. Include a concise fabrication note: board thickness, copper weight (1 oz), soldermask color, controlled impedance requirements (if any)

---

## Quick CvPcb mapping example (CSV-like)

```
Symbol, Suggested footprint
NEO_M8, u-blox_NEO-M8_12x16_LCC
CSI_CONN_22, FPC_22pin_0.5mm_R/A
GPIO_40, PinHeader_2x20_P2.54mm
MICRO_SD, MicroSD_PushPush
REG_3V3, SOT223_AMS1117  # or SOT23-6_MP1584 for switcher
```

---

## Deliverables I prepared in this document

1. Component list + suggested footprints (above)
2. Clear net naming and net class recommendations
3. PCB placement & routing rules for a 2‑layer board
4. Gerber/DRC checklist and test point recommendations

---

## Next steps you can ask me to do now (I can do these directly in KiCad text form):

* I can generate a starter KiCad **schematic sheet** (.kicad_sch) text sketch (symbol instances with net names) you can import.
* I can produce a **CvPcb mapping CSV** for easy assignment.
* I can produce a **starter .kicad_pcb** placement file (component outline + reference) suitable for iterative editing.

If you want any of the above, tell me which option(s) to generate and I will create them right here.

---

## Notes & tips

* If you decide to target a specific SOM (Jetson Nano vs. CM4), choose the matching camera connector early — Pi/CM4 uses the 22‑pin CSI connector; Jetson Nano developer kits typically expose a 15‑pin CSI on a ribbon or have a camera connector depending on the carrier board. Keep both footprints separate and clearly mark one as `OPTIONAL` in the schematic.
* For GNSS antenna use, prefer a u.FL (IPX) connector for external antennas and follow the module manufacturer’s recommended antenna keepout.

