---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Floorplanning step — kết quả của create_floorplan
used_by: [Floorplanning, Placement, MacroPlacement, DesignImport]
requires: [GateLevelNetlist, StandardCell, HardIP, Site]
chain: Chain_PnR_Flow
---
# CoreArea

## Definition
CoreArea là vùng hình chữ nhật bên trong chip nơi tất cả Standard Cells và Macro instances được Placement tool đặt vào. Đây là vùng duy nhất mà PD engineer có thể kiểm soát trực tiếp khi sizing die — các thành phần còn lại (IO Ring, Core-to-IO Spacing) bị quyết định bởi công nghệ và số lượng Pad.

**Cấu trúc phân lớp của Die từ ngoài vào trong:**

$$\text{Die Area} = \text{IO Ring Area} + \text{Core-to-IO Spacing Area} + \text{Core Area}$$

| Vùng | Quyết định bởi |
|---|---|
| IO Ring Area | IO Cell height × số lượng Pad |
| Core-to-IO Spacing | Design Rules (chống latch-up) |
| Core Area | PD Engineer (duy nhất vùng có thể điều chỉnh) |

**Chip Aspect Ratio** — hình dáng của PnR boundary:

$$\text{Aspect Ratio} = \frac{\text{Width}}{\text{Height}}$$

AR = 1.0 → hình vuông. AR > 1.0 → wide. AR < 1.0 → tall. Aspect Ratio được điều chỉnh để khớp với packaging constraints (bond pitch, bump grid) và để bias routing resources theo chiều ưu tiên của dominant data paths.

==Chú ý:== Không có một quy ước “chính thức chung” cho **aspect ratio của core area**; vì vậy không nên hiểu từ `aspect ratio` nếu tài liệu không định nghĩa rõ. Trong ngữ cảnh tổng quát, aspect ratio thường là **width / height**, nhưng trong **OpenROAD/ORFS**, `CORE_ASPECT_RATIO` được định nghĩa là **height / width**.

Aspect Ratio trực tiếp biểu diễn tỉ số tài nguyên routing theo chiều dọc-ngang. Die cao có nhiều horizontal tracks hơn; die rộng có nhiều vertical tracks hơn. Điều này phải được cân nhắc cùng với preferred routing direction của metal stack trong technology.

**Phân loại chip theo bottleneck diện tích:**

| Loại | Đặc trưng | Vấn đề |
|---|---|---|
| Core-limited | Logic lớn (CPU, GPU) → Core quyết định die size | Core quá đặc → congestion |
| Pad-limited | I/O nhiều (Router, Switch) → Pad ring quyết định die size | Core thừa diện tích (lãng phí silicon) |
| Block-limited | Quá nhiều Macros khó xếp chặt | Core Area phình to không cần thiết |
| Package-limited | Package size đặt trần cho die size | Die bị thu nhỏ dưới nhu cầu logic |

## Computed from

**Bước 1 — Ước tính Standard Cell Area từ Synthesis report:**

Lấy tổng Standard Cell area từ Synthesis, sau đó thêm overhead 10–20% cho:
- CTS: Clock Buffers + Clock Inverters được chèn vào sau CTS
- Timing optimization: Upsize, Buffer insertion, ECO spare cells
- Routing congestion relief: White space buffer

**Bước 2 — Tính Core Area từ Target Utilization:**

$$\text{Core Area} = \frac{\text{Estimated Cell Area (SC + Macro)}}{\text{Target Utilization}}$$

Target Utilization được chọn theo mục tiêu QoR và mức độ khó của design. Không đặt 100% vì:
- Routing Tracks cần không gian để đi dây mà không gây congestion
- Optimizer cần white space để chèn Buffers và upsize Cells khi fix timing
- 20–30% white space = "dự trữ kỹ thuật" bắt buộc

**Hai định nghĩa Utilization trong thực tế:**

$$\text{Core Utilization} = \frac{\text{Standard Cell Area} + \text{Macro Area}}{\text{Total Core Area}}$$

$$\text{Standard Cell Utilization} = \frac{\text{Total Standard Cell Area}}{\text{Placeable Standard Cell Area}}$$

Standard Cell Utilization là metric có ý nghĩa thực tế hơn vì loại trừ vùng bị Macro chiếm — đây là số tool Placement thực sự báo cáo (ký hiệu "TU" trong màn hình Innovus).

**Công thức Die Area estimation đầy đủ:**

$$\text{Die Area (mm}^2\text{)} = \frac{\left[\frac{G + T \cdot G + E \cdot G}{D}\right] + I + M}{TU}$$

Trong đó: $G$ = gate count (không bao gồm Macros), $T$ = overhead CTS + timing (10–20%), $E$ = ECO spare overhead, $D$ = gate density (gates/mm² của technology node), $I$ = IO Ring area, $M$ = Macro area, $TU$ = target utilization.

**Giải pháp cho Pad-limited designs:**

Khi số lượng IO Pad quá lớn so với Core logic, các phương pháp thu nhỏ IO Ring area:
- Inline IO Placement: 1 hàng Pad, Bond Pad riêng biệt
- Staggered Non-CUP IO Placement: 2 hàng Pad so le, Bond Pad riêng
- Staggered CUP (Circuit Under Pad) IO Placement: Bond Pad đặt trực tiếp trên IO body — tiết kiệm diện tích nhất
- Flip Chip với RDL (Redistribution Layer): I/O bumps phân bố trên toàn bề mặt, không giới hạn ở periphery

## Constrains
- **[[Placement]]**: Core Area phải đủ lớn để chứa tất cả Standard Cell instances trong Netlist; Standard Cell Utilization > 90% → severe congestion
- **[[Routing]]**: Aspect Ratio ảnh hưởng đến tỉ lệ horizontal/vertical routing resources; tall design cần nhiều vertical layers, wide design cần nhiều horizontal layers
- **[[ClockTreeSynthesis]]**: Core Area lớn → clock tree phải traverse khoảng cách lớn hơn → ClockLatency cao hơn, ClockSkew khó kiểm soát hơn

## Requires
- [[GateLevelNetlist]] — tổng Standard Cell area từ Synthesis report là starting point
- [[StandardCell]] — cell area từ LIB
- [[HardIP]] — Macro area phải được include trong Core Area estimate
- [[Site]] — Core Area boundary phải là bội số của Site dimensions

## Used by
- [[Floorplanning]] — Core Area definition là output chính của sub-step "Define Boundary"
- [[Placement]] — Placement tool pack Standard Cells vào Core Area
- [[MacroPlacement]] — Macros được placed bên trong Core Area

## Key insight
[USER REVIEW — draft suggestion]: Thiết lập sai Core Utilization là một trong những nguyên nhân phổ biến nhất phải "đập đi làm lại" toàn bộ PnR flow. Nếu ép quá cao (>85%), Routing tool sẽ fail sau vài giờ chạy với thông báo "no routing solution" — không có cách fix nào ngoài quay lại Floorplan. Nếu đặt quá thấp (<60%), chip sẽ quá to và đắt. Aspect Ratio không phải là con số tùy ý — nó phải được thống nhất với Package Engineer để đảm bảo các ràng buộc kết nối/package.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Formula: Die Area = IO Ring Area + Core-to-IO Spacing + Core Area
→ Core Area = Estimated Cell Area / Target Utilization
→ Types: Core-limited · Pad-limited · Block-limited · Package-limited
→ Constrains: [[Placement]] · [[Routing]] · [[ClockTreeSynthesis]]
→ Sub-concepts: Aspect Ratio · Core Utilization · Standard Cell Utilization
→ Cùng nhóm: [[Floorplanning]] · [[MacroPlacement]] · [[Site]] · [[Row]]