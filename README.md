# Smart USB Thumb Drive — 64 GB, 4-Layer PCB (KiCad)

A fully routed, manufacture-ready 64 GB USB thumb drive PCB designed from scratch in KiCad. This project demonstrates high-speed digital design, BGA escape routing, impedance-controlled differential pairs, and multi-layer power distribution — all within the physical constraints of a standard USB stick form factor.

## What This Project Is

This is a complete hardware design for a "smart" USB thumb drive built around:

- **RP2040** — Dual-core Arm Cortex-M0+ microcontroller (Raspberry Pi silicon)
- **MTFC64GAPALBH-IT** — 64 GB eMMC NAND flash in a 153-ball TFBGA package (0.5 mm pitch)
- **USB2244** — USB 2.0 to eMMC bridge controller
- **SY8253ADC** — Synchronous buck converter for regulated power
- **LM66100** — Dual ideal-diode OR-ing circuit for power path management
- **INA226** — High-side/low-side current and power monitor (I2C)
- **SSD1306** — 0.91" 128×32 OLED display for status readout
- **TMP119** — Digital temperature sensor
- **TVS diode arrays** — ESD protection on both USB ports

The drive is not just storage — the RP2040 provides programmable logic for monitoring power draw, temperature, and storage status via the OLED, making it a platform for embedded USB experimentation.

## How It Works

### Architecture

The schematic is split into hierarchical sheets for modularity:

| Sheet | Function |
|-------|----------|
| `oring_power` | Dual ideal-diode OR-ing (VBUS vs. external supply) |
| `buck_converter_power` | 3.3 V regulated rail from the OR'd input |
| `usb_bridge` | USB2244 bridge + ESD protection |
| `emmc_storage` | 64 GB eMMC with full 8-bit data bus |
| `rp2040` | MCU core, crystal, SWD, boot circuitry |
| `sensors` | INA226 current monitor + TMP119 temperature sensor |
| `gpio` | RP2040 breakout headers for development |

### PCB Stackup (4 layers)

| Layer | Purpose |
|-------|---------|
| F.Cu | Signal routing (USB diff pairs, eMMC data bus) |
| In1.Cu | Solid ground plane |
| In2.Cu | Power distribution plane |
| B.Cu | Signal routing (secondary) |

The dedicated ground and power planes provide low-impedance return paths and controlled impedance for high-speed signals.

## Key Design Decisions

### 1. BGA Escape Routing (Via-in-Pad)

The 153-ball eMMC package at 0.5 mm pitch cannot be escaped with surface traces alone. I used **via-in-pad** with 0.3 mm diameter / 0.15 mm drill vias placed directly on BGA pads, transitioning signals to inner layers. Custom DRC rules relax annular width minimums (0.075 mm) and clearances specifically within the BGA courtyard to allow this dense routing while maintaining DRC coverage everywhere else.

### 2. Impedance-Controlled USB 2.0 Differential Pairs

USB D+/D− pairs are routed as impedance-controlled differential pairs targeting 90 Ω differential impedance. Trace geometry was calculated using a stack-up impedance tool and verified against the fabricator's capabilities. Tight coupling is maintained throughout the route.

### 3. eMMC Data Bus Length Matching

The 8-bit parallel eMMC data bus is length-matched to ±3 mm (data lines) and ±5 mm (clock), meeting timing requirements for HS400-class operation. KiCad's tuning patterns (meander routing) are applied with 80% corner radius to minimize reflections.

### 4. Dual Ideal-Diode Power OR-ing

Rather than a simple regulator off VBUS, the power architecture uses two TI LM66100 ideal diodes in an OR-ing configuration. This allows the board to be powered from either USB port seamlessly, with near-zero reverse current and minimal voltage drop — critical for a bus-powered device with tight power budgets.

### 5. Custom DRC Rules

Instead of globally weakening design rules to accommodate BGA and fine-pitch ICs, I wrote **per-component DRC exceptions** in KiCad's custom rules language. Each rule is scoped by courtyard (`insideCourtyard`) so that strict defaults (0.1 mm clearance, standard annular widths) apply everywhere except where the design explicitly requires tighter geometry.

### 6. Teardrops and Reliability Features

Teardrops are enabled on all vias, PTH pads, and SMD pads to improve mechanical robustness and reduce stress risers during thermal cycling — important for a device that will be repeatedly plugged and unplugged.

## Repository Structure

```
├── Smart USB Drive - Course.kicad_pro    # KiCad project file
├── Smart USB Drive - Course.kicad_sch    # Top-level schematic
├── Smart USB Drive - Course.kicad_pcb    # PCB layout
├── Smart USB Drive - Course.kicad_dru    # Custom design rules
├── oring_power.kicad_sch                 # Power OR-ing sub-sheet
├── buck_converter_power.kicad_sch        # Buck converter sub-sheet
├── usb_bridge.kicad_sch                  # USB bridge sub-sheet
├── emmc_storage.kicad_sch                # eMMC storage sub-sheet
├── rp2040.kicad_sch                      # RP2040 MCU sub-sheet
├── sensors.kicad_sch                     # Sensors sub-sheet
├── gpio.kicad_sch                        # GPIO breakout sub-sheet
├── Gerbers-Smart_USB_v1.1/               # Production-ready Gerber files
├── Datasheets/                           # Component datasheets
└── Library/                              # Custom symbols, footprints, 3D models
```

## Tools Used

- **KiCad 8** — Schematic capture, PCB layout, DRC, and Gerber generation
- **NextPCB HQDFM** — Design-for-manufacturing verification and impedance stack-up calculation

## Skills Demonstrated

- Multi-layer PCB design with impedance control
- BGA fanout and via-in-pad escape routing
- High-speed differential pair routing (USB 2.0)
- Parallel bus length matching with meander tuning
- Hierarchical schematic design for complex systems
- Power architecture design (ideal-diode OR-ing, buck regulation)
- Custom DRC rule authoring in KiCad
- Full manufacturing output generation (Gerbers, BOM, pick-and-place)
