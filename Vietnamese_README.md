# USB-C to Ethernet (10/100BASE-TX)

> Thiết kế bộ chuyển đổi **USB-C sang Ethernet 10/100 Mbps** sử dụng **LAN9500AI** làm USB 2.0 Ethernet controller/MAC và **ADIN1200** làm Ethernet PHY ngoài.

---

## 1. Tổng quan dự án

Dự án này thiết kế một USB-to-Ethernet adapter với kiến trúc:

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

### Mục tiêu

- Kết nối thiết bị chủ qua **USB-C / USB 2.0**.
- Chuyển đổi USB sang **Ethernet 10/100 Mbps**.
- Sử dụng **ADIN1200 làm PHY Ethernet ngoài** cho LAN9500AI.
- Thiết kế PCB 4 lớp, ưu tiên signal integrity cho USB và Ethernet.
- Hướng tới layout dễ sản xuất và dễ kiểm tra DRC/DFM.

---

## 2. Kiến trúc phần cứng

| Khối | Linh kiện / chức năng |
|---|---|
| USB interface | USB-C receptacle |
| USB-to-Ethernet controller / MAC | Microchip LAN9500AI |
| External Ethernet PHY | Analog Devices ADIN1200 |
| Ethernet magnetics | Würth Elektronik 749020100A |
| Ethernet connector | Amphenol RJHSE-5381 |
| Nguồn chính | USB VBUS |
| Nguồn logic | 3.3 V regulator |
| Ethernet protection | TVS trên các đường MDI |
| Ethernet termination | Bob Smith termination / chassis network |

---

## 3. Schematic

### 3.1 Sơ đồ nguyên lý tổng thể

<!-- Thay đường dẫn bên dưới bằng ảnh schematic thực tế -->
<table>
<tr>
<td align="center" valign="middle" height="320">

**[ THÊM ẢNH SCHEMATIC TỔNG THỂ TẠI ĐÂY ]**

`docs/images/schematic_overview.png`

</td>
</tr>
</table>

### 3.2 Các khối chính

- **USB-C:** nhận USB 2.0 D+/D− và VBUS.
- **LAN9500AI:** xử lý giao tiếp USB 2.0 và Ethernet MAC.
- **ADIN1200:** PHY Ethernet 10/100 Mbps ngoài, giao tiếp với LAN9500AI qua MII.
- **MAG:** biến áp/cách ly Ethernet giữa PHY và cáp mạng.
- **RJ45:** giao tiếp với cáp Ethernet.
- **Power:** tạo các rail nguồn cần thiết cho LAN9500AI và ADIN1200.

---

## 4. Cấu hình LAN9500AI

LAN9500AI được cấu hình để sử dụng **external PHY ADIN1200** thay cho PHY nội bộ.

### Cấu hình chính

```text
PHY_SEL = HIGH
→ Chọn external PHY

PHY_RESET
→ Reset external PHY

PHY_INT
→ Tín hiệu interrupt từ external PHY

MAC ↔ ADIN1200
→ MII
```

### Ghi chú

- `PHY_SEL` là cấu hình quan trọng để LAN9500AI sử dụng PHY ngoài.
- `PHY_RESET` và `PHY_INT` là các đường điều khiển giữa LAN9500AI và ADIN1200.
- Các strap khác của LAN9500AI được đặt theo cấu hình thiết kế hiện tại.

---

## 5. Cấu hình ADIN1200

ADIN1200 được sử dụng như **external 10/100 Ethernet PHY**.

### Cấu hình hiện tại

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

### Các mạch phụ trợ

```text
MDIO
→ 1.5 kΩ pull-up

PHY_RESET
→ 1 kΩ pull-up

REXT
→ 3.01 kΩ → GND
```

> Các giá trị và trạng thái strap cần được đối chiếu với revision datasheet mà dự án sử dụng trước khi sản xuất.

---

# 6. PCB Stackup

PCB sử dụng **4 lớp**:

| Layer | Loại | Chức năng chính |
|---|---|---|
| **L1** | Signal | USB, Ethernet MDI, clock và các signal quan trọng |
| **L2** | Ground Plane | GND reference plane liên tục |
| **L3** | Power Plane | Chủ yếu phân phối +3.3 V |
| **L4** | Signal | MII và các đường signal/phụ trợ khi L1 không đủ chỗ |

### Ý tưởng stackup

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

### Nguyên tắc sử dụng các lớp

**L1 – Signal**

Ưu tiên cho:

- USB D+/D−
- Ethernet MDI differential pairs
- Crystal và các đường XTAL
- Các đoạn MII quan trọng nếu còn không gian

**L2 – GND**

- Giữ càng liên tục càng tốt.
- Là reference/return path chính cho các đường tốc độ cao trên L1.
- Hạn chế tạo split không cần thiết.

**L3 – Power**

- Chủ yếu dùng cho **+3.3 V plane**.
- Các nguồn khác có thể được route trên L4 nếu phù hợp với thiết kế hiện tại.
- Dùng vias để đưa nguồn từ L3 đến các IC/tụ decoupling.

**L4 – Signal**

- MII LAN9500AI ↔ ADIN1200.
- Các tín hiệu control/low-speed còn lại.
- Tránh chạy các đường tốc độ cao dài trên L4 nếu có thể.

---

## 7. PCB Layout

### 7.1 Ảnh PCB 2D

<!-- Thay bằng ảnh PCB thực tế -->
<table>
<tr>
<td align="center" valign="middle" height="320">

**[ THÊM ẢNH PCB 2D TẠI ĐÂY ]**

`docs/images/pcb_2d.png`

</td>
</tr>
</table>

### 7.2 Ảnh PCB 3D

<!-- Thay bằng ảnh PCB thực tế -->
<table>
<tr>
<td align="center" valign="middle" height="320">

**[ THÊM ẢNH PCB 3D TẠI ĐÂY ]**

`docs/images/pcb_3d.png`

</td>
</tr>
</table>

---

## 8. Chiến lược Routing

Thứ tự ưu tiên khi route:

### 1. Ethernet MDI

```text
ADIN1200 → Magnetics → RJ45
```

- `MDI_0_P / MDI_0_N`
- `MDI_1_P / MDI_1_N`
- Route thành differential pairs.
- Mục tiêu impedance: **100 Ω differential**.
- Giữ cặp gần nhau, đường ngắn và tránh stub.

### 2. USB 2.0

```text
USB-C → LAN9500AI
```

- `USB_P / USB_N`
- Route thành differential pair.
- Mục tiêu impedance: **90 Ω differential**.
- Giữ hai đường đi cùng nhau và hạn chế via.

### 3. MII

```text
LAN9500AI ↔ ADIN1200
```

Ưu tiên các nhóm:

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

Nếu L1 không còn đủ chỗ, MII có thể route trên **L4**.

### 4. Crystal

- Đặt crystal thật gần IC.
- Tụ tải crystal đặt gần vùng XTAL.
- Giữ đường XTAL ngắn.
- Hạn chế via trên đường XTAL.

### 5. Nguồn và control

Sau khi hoàn thành các đường critical:

- 3.3 V
- VDDCORE
- RESET
- MDIO/MDC
- LED
- Strap/configuration

---

## 9. Power Distribution

### +3.3 V

Nếu L3 còn đủ diện tích, sử dụng **power plane/polygon +3.3 V** để phân phối nguồn cho các khối chính.

Các nhánh nhỏ có thể kết nối từ pad → via → L3.

```text
L1 / L4
   │
   │ Via
   ▼
L3: +3.3V PLANE
████████████████████████
```

### GND

L2 được giữ làm **GND plane** chính.

Các tụ decoupling cần đường GND ngắn đến L2, ưu tiên đặt via GND gần pad GND của tụ.

---

## 10. Decoupling và Crystal Placement

### LAN9500AI

- Tụ decoupling đặt sát các chân nguồn.
- Các rail PLL/USB/core cần được xử lý theo schematic và datasheet.
- Crystal và tụ crystal đặt gần IC.

### ADIN1200

- AVDD3P3: 100 nF + 10 nF cho từng nguồn/chân theo thiết kế.
- VDDIO: 100 nF + 10 nF cho từng nguồn/chân theo thiết kế.
- `LDO_CAP`: 100 nF xuống GND.
- Tụ center-tap của magnetics: **100 nF** cho từng center tap phía PHY.
- Tụ crystal đặt gần PHY/crystal.

---

## 11. Ethernet Protection

### TVS

Các đường:

```text
MDI_0_P
MDI_0_N
MDI_1_P
MDI_1_N
```

được bảo vệ bằng TVS điện dung thấp.

Mục tiêu:

- Bảo vệ ESD/transient.
- Không làm ảnh hưởng đáng kể đến tín hiệu Ethernet.
- Placement trên PCB phải giữ đường bảo vệ ngắn.

### Magnetics

Sử dụng:

```text
Würth Elektronik 749020100A
```

Magnetics tạo cách ly giữa PHY và cable side.

### RJ45

Sử dụng:

```text
Amphenol RJHSE-5381
```

RJ45 này được sử dụng cùng magnetics rời trong thiết kế hiện tại.

---

## 12. Thiết kế Chassis / Shield

Thiết kế có hai khái niệm ground:

```text
GND
GND_CHASSIS
```

- **GND:** ground của mạch điện tử/PHY.
- **GND_CHASSIS:** ground liên quan đến shield/cable side.

Khu vực RJ45, shield và Bob Smith termination được bố trí theo topology Ethernet của thiết kế.

---

## 13. DRC / Verification

Các hạng mục kiểm tra chính:

- Clearance
- Width
- Via / hole size
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

### Mục tiêu

Trước khi xuất Gerber/production files:

```text
ERC / schematic check
        ↓
PCB DRC
        ↓
Connectivity check
        ↓
DFM review
        ↓
Gerber / NC Drill
        ↓
Manufacturing
```

---

## 14. Files

Cấu trúc repository đề xuất:

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

> Nên lưu phiên bản datasheet đã dùng cho thiết kế vào repository để tránh thay đổi tài liệu giữa các lần revision.

---

## 16. Design Status

| Hạng mục | Trạng thái |
|---|---|
| Schematic | 🟡 Đang hoàn thiện |
| LAN9500AI configuration | 🟡 Đã cấu hình external PHY |
| ADIN1200 configuration | 🟡 Đã cấu hình MII |
| PCB placement | 🟡 Đang hoàn thiện |
| USB routing | 🟡 Đang kiểm tra |
| Ethernet routing | 🟡 Đang kiểm tra |
| Power plane | 🟡 Đang hoàn thiện |
| DRC | 🟡 Đang xử lý violations |
| Manufacturing files | ⚪ Chưa xuất |
| Prototype | ⚪ Chưa sản xuất |

---

## 17. Revision History

| Revision | Ngày | Nội dung |
|---|---|---|
| A | YYYY-MM-DD | Phiên bản thiết kế ban đầu |
| B | YYYY-MM-DD | |
| C | YYYY-MM-DD | |

---

## 18. Tác giả

**Thiết kế:** Hiep Hoang

**Tên dự án:** USB-C to Ethernet (10/100BASE-TX)

**CAD:** Altium Designer

**PCB Stackup:** 4 Layers

**Revision:** A
