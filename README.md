# USB-C to Ethernet (10/100BASE-TX)

> A USB-C to 10/100 Mbps Ethernet adapter based on the **Microchip LAN9500AI** USB 2.0 Ethernet controller/MAC and the **Analog Devices ADIN1200** external Ethernet PHY.

---

## 1. Project Overview

This project is a USB-to-Ethernet adapter with the following architecture:

```text
USB-C
  │
  │ USB 2.0 D+/D-
  ▼
LAN9500AI
USB 2.0 Ethernet Controller / MAC
  │
  │ MII
  ▼
ADIN1200
10/100 Ethernet PHY
  │
  │ MDI differential pairs
  ▼
Ethernet Magnetics
749020100A
  │
  ▼
RJ45
RJHSE-5381
  │
  ▼
Ethernet Cable
```

### Project Goals

- Provide USB 2.0 connectivity through a USB-C connector.
- Convert USB 2.0 data to **10/100 Mbps Ethernet**.
- Use **ADIN1200 as the external Ethernet PHY** for LAN9500AI.
- Use a 4-layer PCB stackup with controlled routing for USB and Ethernet signals.
- Keep the design manufacturable and suitable for DRC/DFM verification.

---

## 2. Hardware Architecture

| Block | Component / Function |
|---|---|
| USB interface | USB-C receptacle |
| USB-to-Ethernet controller / MAC | Microchip LAN9500AI |
| External Ethernet PHY | Analog Devices ADIN1200 |
| Ethernet magnetics | Würth Elektronik 749020100A |
| Ethernet connector | Amphenol RJHSE-5381 |
| Main power source | USB VBUS |
| Logic supply | 3.3 V regulator |
| Ethernet protection | Low-capacitance TVS on MDI lines |
| Ethernet termination | Bob Smith termination / chassis network |

---

## 3. Schematic

### 3.1 Overall Schematic

<!-- Replace the placeholder below with the actual schematic image -->
<table>
<tr>
<td align="center" valign="middle" height="320">

<img width="1218" height="791" alt="image" src="https://github.com/user-attachments/assets/640bdcd9-e1e6-4ea0-b28d-c29c6a989272" />


`docs/images/schematic_overview.png`

</td>
</tr>
</table>

### 3.2 Main Functional Blocks

- **USB-C:** USB 2.0 D+/D− and VBUS interface.
- **LAN9500AI:** USB 2.0 Ethernet controller and MAC.
- **ADIN1200:** External 10/100 Mbps Ethernet PHY connected through MII.
- **MAG:** Ethernet transformer/magnetics providing galvanic isolation to the cable side.
- **RJ45:** Physical Ethernet cable interface.
- **Power:** Generates the required supply rails for LAN9500AI and ADIN1200.

---

## 4. LAN9500AI Configuration

LAN9500AI is configured to use **ADIN1200 as the external Ethernet PHY** instead of its internal PHY.

### Main Configuration

```text
PHY_SEL = HIGH
→ Select external PHY (ADIN1200)

PHY_RESET
→ Reset external PHY

PHY_INT
→ Interrupt signal from external PHY

MAC ↔ ADIN1200
→ MII interface
```

### Notes

- `PHY_SEL` is the main hardware selection for external PHY operation.
- `PHY_RESET` is used to reset the external PHY.
- `PHY_INT` provides the interrupt connection between the PHY and LAN9500AI.
- Other LAN9500AI straps are set according to the intended hardware configuration.

---

## 5. ADIN1200 Configuration

ADIN1200 is used as the **external 10/100 Mbps Ethernet PHY**.

### Current Hardware Strap Configuration

```text
RXCLK = HIGH
RXDV  = LOW
→ MII MODE

RXD0 = HIGH
RXD1–RXD3 = LOW
→ PHY ADDRESS = 0x01

MAC_COL = HIGH
PHY_LINK_ST = LOW
→ PHY hardware configuration strap
```

### Supporting Circuits

```text
MDIO
→ 1.5 kΩ pull-up

PHY_RESET
→ 1 kΩ pull-up

REXT
→ 3.01 kΩ → GND
```

> Strap values and functions should be checked against the exact datasheet revision used for production.

---

# 6. PCB Stackup

The PCB uses a **4-layer stackup**:

| Layer | Type | Main Function |
|---|---|---|
| **L1** | Signal | USB, Ethernet MDI, clock, and other critical signals |
| **L2** | Ground Plane | Solid GND reference / return plane |
| **L3** | Power Plane | Mainly +3.3 V distribution |
| **L4** | Signal | MII and lower-priority signal routing |

### Stackup Concept

```text
┌─────────────────────────────┐
│ L1  SIGNAL                  │
├─────────────────────────────┤
│ L2  GND PLANE               │
├─────────────────────────────┤
│ L3  POWER / +3.3V           │
├─────────────────────────────┤
│ L4  SIGNAL                  │
└─────────────────────────────┘
```

### Layer Usage Strategy

**L1 – Signal**

Priority routing for:

- USB D+/D−
- Ethernet MDI differential pairs
- Crystal / XTAL traces
- Critical MII traces where space allows

**L2 – GND**

- Keep as continuous as practical.
- Provides the main reference and return path for L1 signals.
- Avoid unnecessary splits beneath high-speed routing.

**L3 – Power**

- Used mainly as a **+3.3 V power plane**.
- Other supply rails can be routed on L4 where appropriate.
- Vias are used to connect L3 power to ICs and decoupling capacitors.

**L4 – Signal**

- Mainly used for LAN9500AI ↔ ADIN1200 MII.
- Used for control and lower-priority signals when L1 is congested.
- Avoid moving critical high-speed signals to L4 unless necessary.

---

## 7. PCB Layout

### 7.1 2D PCB Layout

<!-- Replace the placeholder below with the actual PCB image -->
<table>
<tr>
<td align="center" valign="middle" height="320">

<img width="1356" height="591" alt="image" src="https://github.com/user-attachments/assets/9201e9d1-c8b8-48d6-9bf1-a5bceab7435c" />
<img width="1330" height="621" alt="image" src="https://github.com/user-attachments/assets/2a44d273-2e52-4666-972c-61a5ed47cc22" />
<img width="1276" height="577" alt="image" src="https://github.com/user-attachments/assets/44260860-9651-4d01-a8c0-5f414acc4ba1" />
<img width="1317" height="591" alt="image" src="https://github.com/user-attachments/assets/74f3b306-c23b-49d1-b437-03e7848ccbf6" />


`docs/images/pcb_2d.png`

</td>
</tr>
</table>

### 7.2 3D PCB View

<!-- Replace the placeholder below with the actual 3D image -->
<table>
<tr>
<td align="center" valign="middle" height="320">

<img width="913" height="410" alt="image" src="https://github.com/user-attachments/assets/14fba79f-b388-4785-901b-62913fd11330" />


`docs/images/pcb_3d.png`

</td>
</tr>
</table>

---

## 8. Routing Strategy

### 1. Ethernet MDI

```text
ADIN1200 → Magnetics → RJ45
```

Signals:

- `MDI_0_P / MDI_0_N`
- `MDI_1_P / MDI_1_N`

Routing goals:

- Route as differential pairs.
- Target **100 Ω differential impedance**.
- Keep each pair closely coupled.
- Keep the PHY-to-magnetics path short.
- Avoid unnecessary vias and stubs.

### 2. USB 2.0

```text
USB-C → LAN9500AI
```

Signals:

- `USB_P / USB_N`

Routing goals:

- Route as a differential pair.
- Target **90 Ω differential impedance**.
- Keep D+ and D− closely coupled.
- Minimize vias and stubs.
- Keep the path clean and away from noisy switching nodes.

### 3. MII

```text
LAN9500AI ↔ ADIN1200
```

Main signals:

```text
TXD0..TXD3
TX_CLK
TX_EN

RXD0..RXD3
RX_CLK
RX_DV
RX_ER

MDC
MDIO
CRS
COL
```

If L1 routing becomes congested, MII can be routed on **L4**.

### 4. Crystal Routing

- Place each crystal close to its IC.
- Place crystal load capacitors close to the IC/crystal network.
- Keep XTAL traces short.
- Avoid vias on XTAL traces where possible.
- Keep crystals away from switching regulators and noisy high-speed traces.

### 5. Power and Control Signals

After critical routing is complete, route:

- +3.3 V distribution
- VDDCORE
- RESET
- MDIO / MDC
- LED signals
- Hardware straps / configuration signals

---

## 9. Power Distribution

### +3.3 V

If sufficient L3 area is available, use **L3 as a +3.3 V power plane/polygon**.

Small local branches can be connected from:

```text
L1 / L4
   │
   │ Via
   ▼
L3: +3.3V PLANE
████████████████████████
```

The main goal is to keep power distribution low impedance and keep decoupling connections short.

### GND

L2 is used as the main **GND plane**.

For decoupling capacitors:

- Keep the capacitor close to the IC power pin.
- Keep the GND connection short.
- Use nearby GND vias to connect to L2.

---

## 10. Decoupling and Crystal Placement

### LAN9500AI

- Place decoupling capacitors close to the corresponding power pins.
- Handle PLL, USB, and core supply decoupling according to the datasheet.
- Place the crystal and crystal capacitors close to the IC.

### ADIN1200

- AVDD3P3: 100 nF + 10 nF for each supply connection as implemented.
- VDDIO: 100 nF + 10 nF for each supply connection as implemented.
- `LDO_CAP`: 100 nF to GND.
- Magnetics center-tap capacitors: **100 nF** for each PHY-side center tap.
- Crystal load capacitors placed close to the PHY/crystal network.

---

## 11. Ethernet Protection

### TVS Protection

The following MDI lines are protected:

```text
MDI_0_P
MDI_0_N
MDI_1_P
MDI_1_N
```

The TVS devices are intended to provide:

- ESD protection.
- Transient protection.
- Minimal added capacitance to the Ethernet signal path.

TVS placement should keep the protection path short and low inductance.

### Magnetics

Current design uses:

```text
Würth Elektronik 749020100A
```

The magnetics provide galvanic isolation between the PHY and the Ethernet cable side.

### RJ45

Current design uses:

```text
Amphenol RJHSE-5381
```

The RJ45 connector is used together with the external magnetics in this design.

---

## 12. Chassis / Shield Design

The design separates:

```text
GND
GND_CHASSIS
```

- **GND:** electronic circuit / PHY ground.
- **GND_CHASSIS:** cable-side / shield-related ground.

The RJ45 shield, Bob Smith termination, and chassis network are handled separately from the PHY signal ground according to the intended Ethernet topology.

---

## 13. DRC / Verification

Main checks include:

- Clearance
- Width
- Via and hole size
- Silkscreen clearance
- Solder-mask sliver
- Differential-pair gap
- Differential-pair uncoupled length
- Length matching
- Room / placement constraints
- Net connectivity
- Unrouted nets
- GND plane connectivity
- Power-plane connectivity

### Verification Flow

```text
Schematic / ERC
       ↓
PCB DRC
       ↓
Connectivity Check
       ↓
DFM Review
       ↓
Gerber / NC Drill
       ↓
Manufacturing
```

---

## 14. Project Files

Suggested repository structure:

```text
.
├── README.md
├── USBc-to-Ethernet.PrjPcb
├── Schematic/
│   └── USBc-to-Ethernet.SchDoc
├── PCB/
│   └── USBc-to-Ethernet.PcbDoc
├── Libraries/
├── Manufacturing/
│   ├── Gerber/
│   ├── NC_Drill/
│   └── Pick_and_Place/
├── docs/
│   └── images/
│       ├── schematic_overview.png
│       ├── pcb_2d.png
│       └── pcb_3d.png
└── Datasheets/
```

---

## 15. Datasheets / References

- Microchip **LAN9500AI**
- Analog Devices **ADIN1200**
- Würth Elektronik **749020100A**
- Amphenol **RJHSE-5381**
- USB Type-C / USB 2.0 design references

> Keep the exact datasheet revisions used for the design in the repository so that future revisions can be traced back to the original design requirements.

---

## 16. Design Status

| Item | Status |
|---|---|
| Schematic | 🟡 In progress |
| LAN9500AI configuration | 🟡 External PHY configuration implemented |
| ADIN1200 configuration | 🟡 MII configuration implemented |
| PCB placement | 🟡 In progress |
| USB routing | 🟡 Under review |
| Ethernet routing | 🟡 Under review |
| Power plane | 🟡 In progress |
| DRC | 🟡 Violations being resolved |
| Manufacturing files | ⚪ Not generated |
| Prototype | ⚪ Not fabricated |

---

## 17. Revision History

| Revision | Date | Description |
|---|---|---|
| A | 2026-09-21 | Initial design |
| B | YYYY-MM-DD | |
| C | YYYY-MM-DD | |

---

## 18. Author

**Designer:** Hiep Hoang

**Project:** USB-C to Ethernet (10/100BASE-TX)

**CAD Tool:** Altium Designer

**PCB Stackup:** 4 Layers

**Revision:** A
