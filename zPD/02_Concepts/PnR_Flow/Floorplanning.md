---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Bước thứ hai trong PnR flow — sau DesignImport, trước Placement; output lưu dưới dạng DEF
used_by: [Placement, ClockTreeSynthesis, Routing, PowerAnalysis]
requires: [DesignImport, GateLevelNetlist, SDC, LEF, CoreArea, IOCell, HardIP, PlacementBlockage, Row]
chain: Chain_PnR_Flow
---
# Floorplanning

## Definition
Floorplanning là bước major đầu tiên trong PD flow sau DesignImport, đồng thời là điểm chuyển giao mang tính vĩ mô từ không gian logic (Netlist) sang không gian vật lý (Silicon). Chất lượng Floorplan quyết định sự thành bại của tất cả các bước phía sau (Placement, CTS, Routing) — một Floorplan tệ không thể được optimizer cứu vớt, toàn bộ flow phải iterate lại từ đầu.

Floorplanning diễn ra ở hai cấp độ thiết kế:

**Full-chip Floorplan:** PD engineer tự quyết định die dimensions, IO Pad positions, Macro positions. Core Area là vùng duy nhất có thể kiểm soát trong sizing.

**Block-level Floorplan:** Block designer nhận PR boundary từ top-level integration — không tự định nghĩa. Block boundary thường là rectilinear (hình chữ L, T) để fit vào chip topology. IO Pins của block được kế thừa từ top-level và phải honored exactly.

**6 mục tiêu của Floorplanning:**

Từ sau DesignImport, Floorplan phải đồng thời đạt được: (1) Define optimal physical boundaries phù hợp packaging constraints, (2) Optimize PPA trên toàn design, (3) Enable clean and legal IO placement để support top-level integration, (4) Place Macros và define SC areas cho best timing + utilization + routing efficiency, (5) Build robust Power Delivery Network (PDN) đạt IR Drop + EM requirements, (6) Create custom pre-routes cho critical/sensitive nets khi cần (clock, reset, analog signals).

## Computed from

Floorplanning là quyết định kỹ thuật, không phải computation thuần túy. Core Area ước tính được tính từ:

$$\text{Core Area} = \frac{\text{Total Standard Cell Area} + \text{Macro Area}}{\text{Target Utilization}}$$

Target Utilization điển hình: **70–80%**. Phần white space còn lại là bắt buộc — để chừa routing Tracks và để optimizer chèn Buffers khi fix timing.

**7 sub-steps của Floorplanning** (theo thứ tự):

| Sub-step | Nội dung | Output |
|---|---|---|
| 1. Define Boundary | Xác định Core Area, Die size, Aspect Ratio, Core Utilization target | Core boundary + Row definitions |
| 2. Place IOs | Place IO Pads (full-chip) hoặc assign Block Pins (block-level) | IO Cell positions / Pin locations |
| 3. Place Macros | Macro Placement dựa trên flight-line analysis + 7 key factors | FIXED Macro positions + Halos |
| 4. Define SC Areas | Xác định vùng Standard Cell placement; insert Pre-placed Cells | Placeable SC areas + Physical-only cells |
| 5. Build Power Grid | Tạo PDN: power rings → power straps → Standard Cell rail connections | PDN structure |
| 6. Check Quality | Verify congestion (trial routing), timing, power (IR Drop estimate) | Floorplan QoR report |
| 7. Go to Placement | Exit criteria pass → proceed | Validated design DB → [[Placement]] |

Sau Check Quality, nếu có violations, engineer quay lại bất kỳ sub-step nào để fix. Iteration loop phổ biến nhất: MacroPlacement → Check Quality → adjust → repeat.

**Pre-placed Cells (sub-step 4) — 3 loại bắt buộc:**

*Well-Tap Cells*: Ngăn Latch-up Effect — cấu trúc N-well/P-substrate trong CMOS tạo ra BJT parasitic; nhiễu điện áp có thể trigger tạo dòng đoản mạch. Tap Cells nối N-well → VDD và P-sub → VSS. Foundry DRC quy định khoảng cách tối đa giữa Tap Cells — vi phạm = fab rejection.

*End-Cap Cells (Boundary Cells)*: Đặt ở hai đầu mỗi Placement Row. Standard Cells ở rìa Row bị sai số hình học lithography do thiếu neighboring cells làm bệ đỡ. End-cap Cells lấp vào rìa để bảo vệ logic thực và ngắt mạch an toàn cho Power Rails ở cuối Row.

*Decap Cells*: Chống Dynamic IR Drop. Khi hàng triệu Flip-flops chuyển trạng thái cùng lúc tại clock edge, dòng điện rút ra đột ngột. Decap Cells hoạt động như "bể chứa điện cục bộ" — cung cấp instantaneous current để giữ VDD ổn định trong nanoseconds đầu sau clock edge.

**Netlist Uniquification:**

Trước khi bắt đầu Floorplan optimization, PnR tool tạo bản sao vật lý độc lập cho tất cả instances của cùng một module (ví dụ: `ALU_1` và `ALU_2` từ cùng module `ALU`). Nếu không uniquify, khi tool chèn Buffer vào `ALU_1` để fix Setup violation, nó sẽ apply thay đổi đó cho cả `ALU_2` — phá vỡ timing của `ALU_2`. Uniquification cắt đứt shared resource → local optimization an toàn.

## Constrains
- **[[Placement]]**: Core boundary, Macro FIXED positions, Row definitions, Placeable SC areas, Blockages — tất cả là hard constraints cho Placement engine; Standard Cells chỉ được placed trong Core area còn lại
- **[[ClockTreeSynthesis]]**: khoảng cách vật lý giữa clock source và clock sinks (đặc biệt clocked Macros) ảnh hưởng trực tiếp đến [[ClockSkew]] và [[ClockLatency]]
- **[[Routing]]**: PDN chiếm routing resources trên upper metal layers; Macro blockages và Routing Blockages định hình routing channels; wire length estimates giữa blocks ảnh hưởng inter-block Timing closure
- **[[PowerAnalysis]]** (IR Drop): topology và width của power stripes trong PDN từ Floorplanning quyết định voltage drop toàn chip; PDN sau Floorplan là input chính cho IR Drop verification

## Requires
- [[DesignImport]] — validated design database là prerequisite
- [[GateLevelNetlist]] — xác định số lượng cells, hierarchy, connectivity để ước tính area và quyết định Macro placement
- [[SDC]] — Timing constraints xác định critical paths, quyết định vị trí tương đối của blocks
- [[LEF]] — Cell Abstracts (kích thước Macro, Pin locations, Obstruction), Site/Row definitions, Track definitions
- [[CoreArea]] — die sizing và utilization calculation
- [[IOCell]] — IO Pad placement trong sub-step 2
- [[HardIP]] — Macro placement trong sub-step 3
- [[PlacementBlockage]] — congestion management quanh Macros
- [[Row]] — Standard Cell placement infrastructure

## Used by
- **[[Placement]]** — nhận Core area và Macro FIXED positions làm hard constraints, place Standard Cells vào phần còn lại
- **[[ClockTreeSynthesis]]** — nhận physical locations của clock sinks để build Clock Tree
- **[[Routing]]** — nhận PDN routes từ Floorplanning, route signal nets trong routing tracks còn lại
- **[[PowerAnalysis]]** (IR Drop) — verify PDN structure từ Floorplanning đủ robust

## Key insight
[USER REVIEW — draft suggestion]: Floorplanning là bước có tác động lan truyền lớn nhất và leverage cao nhất trong toàn bộ PD flow: một quyết định sai ở đây (Macro placement tạo routing congestion, PDN thiếu width, Core area quá chật) không thể được cứu bởi Placement hay Routing tốt về sau. Nguyên tắc thực tế: "Think routing first, place cells second" — mọi quyết định Floorplan phải hỏi "Router có đủ không gian để đi dây ở đây không?" trước khi hỏi "Cell này có đúng vị trí về timing không?". Một Floorplan routing-friendly sẽ giải quyết phần lớn timing violations một cách tự nhiên; một Floorplan timing-focused nhưng routing-hostile sẽ fail ở Routing dù timing trông đẹp trên paper.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ 7 sub-steps: Define Boundary → Place IOs → Place Macros → Define SC Areas → Build Power Grid → Check Quality → Go to Placement
→ 6 goals: Boundaries · PPA · IO Placement · Macro + SC Areas · PDN · Critical Pre-routes
→ Pre-placed Cells: Tap Cells (latch-up) · End-cap Cells (lithography) · Decap Cells (IR Drop)
→ Uniquification: prerequisite for safe local optimization
→ Inputs: [[DesignImport]] · [[GateLevelNetlist]] · [[SDC]] · [[LEF]] · [[CoreArea]] · [[IOCell]] · [[HardIP]]
→ Outputs → [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[PowerAnalysis]]
→ Cùng nhóm: [[CoreArea]] · [[MacroPlacement]] · [[PlacementBlockage]] · [[RoutingBlockage]] · [[IOCell]]