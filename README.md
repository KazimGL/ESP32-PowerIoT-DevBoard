# ESP32 PowerIoT Dev Board

**A battery-powered ESP32 development board with onboard Li-ion charging, protection, regulation, and sensor support — designed end-to-end in KiCad.**

Designed by **Kazim Bhojani** ([KaizBuilts](#)) — a fresher PCB design project built to learn the complete hardware design workflow from schematic to fabrication-ready Gerbers.

---

## 📋 Overview

This board combines four functional subsystems into a single 2-layer PCB:

| Block | Function |
|---|---|
| **Power** | USB-C input, ESD protection, TP4056 Li-ion charging, DW01A + FS8205A battery protection, AP2112K-3.3 regulation, battery voltage sensing |
| **Processor** | ESP32-WROOM-32E module with reset/boot circuitry |
| **Communication** | CH340C USB-UART bridge with auto-program (DTR/RTS reset) circuit, I2C breakout |
| **I/O** | Dual GPIO breakout headers (1×15 each), user LED, user button, BME280 environmental sensor |

---

## ⚙️ Specs

- **MCU:** ESP32-WROOM-32E (WiFi + Bluetooth)
- **Charging:** TP4056, USB-C input, configurable charge current
- **Protection:** DW01A + FS8205A (overcharge / overdischarge / short-circuit)
- **Regulation:** AP2112K-3.3, 600mA LDO
- **Sensor:** BME280 (temperature, humidity, pressure) over I2C
- **Layers:** 2-layer PCB
- **Design tool:** KiCad 10.0.5
- **Board size:** ~65mm × 50mm

---

## 🖼️ Images

**PCB Layout / Routing**
![PCB Routing](Screenshot%202026-10-06%20162758.png)

**3D Render**
![PCB 3D Render](Screenshot%202026-10-06%20162957.png)

**Schematic**
📄 [schematic.pdf](schematic.pdf)

**Full Project Document**
📄 [PRoject_cv.pdf](PRoject_cv.pdf)

---

## 🧠 Design Process & AI-Assisted Workflow

This project was built while learning PCB design fundamentals from scratch, using **Claude (Anthropic)** as a design-review assistant throughout the process — not to generate the design automatically, but to:

- Review each schematic block (Power, Processor, Communication, I/O) for wiring correctness before moving forward
- Explain *why* certain circuits work the way they do (e.g., the EN/IO0 auto-reset logic, battery protection MOSFET gate drive, I2C pull-up sizing)
- Walk through ERC/DRC error messages line-by-line and explain root causes rather than just handing over a fix
- Catch real mistakes along the way (several were my own, and at least one was the AI's — an incorrect DTR/RTS pin mapping that was caught and corrected by cross-checking against esptool's actual reset sequence rather than assumption)

This was an intentionally iterative process: wire a section → verify in ERC → get it reviewed → fix → repeat. Every connection in this board was manually drawn, checked, and understood — the AI's role was review and explanation, not auto-generation.

---

## 🐛 Real Issues Faced During This Build

Documenting these because they were the most time-consuming (and most educational) parts of the project:

1. **KiCad global symbol library table accidentally broken** — while trying to fix a missing footprint library path, the entire global symbol library table collapsed down to a single broken entry, causing 72 cascading ERC warnings across unrelated components (`power`, `Diode`, `Switch`, `Connector` libraries all failed to load). Fixed using KiCad's **"Reset Libraries"** button in the Symbol Libraries manager.

2. **Custom part (FS8205A) had no native KiCad footprint** — imported via `easyeda2kicad` from its LCSC part number, which required linking both a symbol library *and* a separate footprint library (`.pretty` folder) through two different dialogs in two different KiCad editors (Schematic Editor vs. PCB Editor) — not obvious at first.

3. **Duplicate global labels merging unrelated nets** — reused generic labels like `INPUT` and `OUTPUT` in multiple places on the schematic, which KiCad silently merged into a single net across the whole sheet, causing misleading ERC warnings until traced back and renamed to unique, meaningful net names (`VBUS_FUSED`, `3V3`, `-BATT`, etc).

4. **USB-C footprint/symbol pin-count mismatch** — J1 was assigned a 6-pin "power only" USB-C footprint while the schematic symbol used the full 16-pin USB 2.0 pinout (including D+/D−), causing 10 pad-not-found errors during PCB sync. Fixed by selecting the correct 16-pin footprint (`USB_C_Receptacle_HRO_TYPE-C-31-M-12`) matching the real connector's datasheet.

5. **Invalid reference designators blocking PCB sync** — two LED symbols were accidentally named `Green(Done)1` and `RED(charging)1` instead of standard alphanumeric designators, which KiCad's schematic/ERC tolerated but the PCB sync step rejected outright. Renamed to proper `D4`/`D5` format.

6. **DRC hole-size violation on the ESP32 module footprint** — the ESP32-WROOM-32E footprint's castellated pads required a 0.2mm minimum drill size, but the board's design rules were set to a 0.3mm minimum, causing 21 repeated violations. Resolved by adjusting the board's minimum drill size constraint to match both the footprint and the target fab's actual capability.

7. **Incomplete thermal relief connections** on copper pour zones for small shield/ground pads — fixed by reducing the thermal relief gap and spoke width settings to better fit smaller pad geometries.

Each of these was worked through methodically: understand the error → identify root cause → fix → re-verify, rather than blindly applying fixes.

---

## 📁 Repository Contents

```
project_CV.kicad_pro    → KiCad project file
project_CV.kicad_sch    → Schematic source
project_CV.kicad_pcb    → PCB layout source
schematic.pdf           → Exported schematic (all 4 blocks)
PRoject_cv.pdf          → Full project document
*.png                   → PCB layout / 3D render screenshots
README.md               → This file
```

---

## 🔧 Tools Used

- **KiCad 10.0.5** — schematic capture, PCB layout, DRC/ERC
- **easyeda2kicad** — custom part import (FS8205A battery protection MOSFET)
- **Claude (Anthropic)** — design review and debugging assistant

---

---

*Built as a learning project to understand the full PCB design lifecycle — schematic capture, component selection, DFM, fabrication file generation, and bring-up — end to end.*
