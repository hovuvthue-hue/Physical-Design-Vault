---
tags: [concept, cell-library]
group: Cell Library
defined_in: Standard Cell Library — provided by Foundry hoặc Library vendor; characterized cho specific technology node
used_by: [LogicSynthesis, Placement, Routing, ClockTreeSynthesis, STA, Signoff]
requires: [LEF, LIB, GDS, Site]
chain: Chain_LEF_to_PnR
---
# StandardCell

## Definition
Standard Cell (còn gọi là **leaf cell** trong ngữ cảnh Netlist và Import) là một logic function được thiết kế sẵn, pre-characterized, và tuân thủ một tập constraints vật lý cố định: chiều cao uniform bằng Site height, chiều rộng là bội số của Site width, và VDD/VSS power rails ở vị trí cố định trên/dưới. Tính "standard" nằm ở chỗ tất cả cells trong cùng một library đều chia sẻ cùng chiều cao và power rail positions — cho phép xếp liền kề trong Rows mà không cần special treatment.

Standard Cells là building blocks nhỏ nhất trong digital design. Phân biệt rõ với Macros: Macros (SRAM, PLL, PHY) không phải Standard Cells — chúng lớn hơn nhiều, có fixed layout, và không phải leaf cells trong Netlist hierarchy.

Standard Cell library là foundation của toàn bộ digital design flow: Logic Synthesis map RTL về Standard Cells, Placement đặt chúng trên silicon, Routing kết nối chúng.

## Computed from
Standard Cell không được compute trong EDA flow mà là pre-designed artifact. Mỗi Standard Cell được cung cấp với đầy đủ views:

- **LEF (.lef)**: Cell Abstract (PR Boundary + Pins + OBS) — dùng cho Placement và Routing
- **LIB (.lib)**: timing/power model per PVT corner — dùng cho STA và Logic Synthesis
- **GDS (.gds)**: full layout — dùng cho DRC/LVS và Tape-out
- **Verilog (.v)**: functional model — dùng cho logic simulation
- **SPICE/CDL**: transistor-level netlist — dùng cho electrical simulation và LVS

**Drive Strength variants**: cùng logic function, nhiều drive strengths — INVX1 (nhỏ/yếu) đến INVX4 (lớn/mạnh). Drive strength lớn hơn = transistor width $W$ lớn hơn = $I_{ON}$ lớn hơn = charge load nhanh hơn = Cell Delay nhỏ hơn, nhưng đồng thời = diện tích lớn hơn + input capacitance lớn hơn + leakage power lớn hơn.

**Multi-Vth variants**: cùng footprint (identical SIZE), khác threshold voltage:

| Variant | Speed | Leakage | Use case |
|---|---|---|---|
| HVT (High Vth) | Chậm | Thấp | Non-critical paths — tiết kiệm power |
| RVT/SVT (Regular/Standard Vth) | Trung bình | Trung bình | Default |
| LVT (Low Vth) | Nhanh | Cao | Critical paths — đáp ứng timing |

Vth optimization (HVT → LVT swap trên critical paths) là một trong những lever mạnh nhất của PD engineer để cùng lúc optimize power và timing mà không thay đổi floorplan.

**Physical Cell variants** (không có logic function, nhưng bắt buộc trong physical implementation):

| Type | Chức năng | Khi dùng |
|---|---|---|
| Filler Cell | Lấp khoảng trống trong Rows; đảm bảo N-Well và VDD/VSS rails liên tục | Sau Placement để fill gaps |
| Tap Cell / Well Tap | Kết nối N-Well với VDD và P-Substrate với VSS → prevent latch-up | Định kỳ trong Rows |
| Decap Cell | Tụ điện decoupling cục bộ → stabilize power supply, giảm IR Drop noise | Trong Core area |
| End Cap Cell | Đặt ở hai đầu mỗi Row → fix lithography edge effects | Mỗi Row boundary |
| ECO Cell | Spare logic cells đặt sẵn → enable metal-only ECO fix sau Signoff | Pre-placed trong design |

## Constrains
- **[[Placement]]**: StandardCell là đối tượng movable chính trong placement stage; tất cả Standard Cells phải được đặt hợp lệ trong [[Row]] với [[Site]]-aligned coordinates trên [[PlacementGrid]]. Uniform height + variable width (theo bội số Site width) là điều kiện nền để legal placement nhất quán.
- **[[Routing]]**: Standard Cell Pins phải align với Routing Grid; internal OBS giới hạn over-cell routing; cell drive strength ảnh hưởng đến Net Delay (weak driver → high output resistance → slow RC charging)
- **[[STA]]**: drive strength và Vth variant của từng cell trực tiếp ảnh hưởng đến Cell Delay (via LIB LUT lookup); Vth swapping là common ECO để fix Setup violations trên critical paths
- **[[LogicSynthesis]]**: Technology Mapping = matching generic Boolean logic với Standard Cell functions trong library; synthesis chọn drive strength và Vth variants dựa trên timing/area/power objectives

## Requires
- [[LEF]] — Macro LEF (Cell Abstract) là physical representation dùng trong PnR
- [[LIB]] — Liberty file chứa timing/power characterization data tại từng PVT corner
- [[GDS]] — full layout dùng cho DRC/LVS verification và Tape-out
- [[Site]] — Standard Cell phải comply với Site height và width constraints

## Used by
- [[LogicSynthesis]] — maps RTL logic functions → Standard Cell instances trong Technology Mapping; selects drive strength và Vth variants
- [[GateLevelNetlist]] — leaf cells tạo thành building blocks chính của Netlist
- [[Placement]] — places Standard Cell instances lên Placement Grid trong Rows
- [[Routing]] — routes signal nets giữa Standard Cell Pins
- [[ClockTreeSynthesis]] — uses special Clock Cell variants (balanced rise/fall delay) để build Clock Tree
- [[STA]] · [[Signoff]] — reads LIB timing data của từng cell để compute Cell Delay và Slack

## Key insight
[USER REVIEW — draft suggestion]: Standard Cell library là ranh giới giữa thế giới lý thuyết (Boolean logic, RTL) và thế giới vật lý (silicon dimensions, transistor physics). Mọi thứ phía trên Standard Cell là technology-independent; mọi thứ phía dưới là technology-specific. Đây là lý do khi chuyển sang process node mới (28nm → 7nm), PD team phải re-synthesize và re-place-route toàn bộ — không thể reuse physical layout vì Standard Cell dimensions, Pitch, và electrical characteristics hoàn toàn khác. Physical Cell variants (Filler, Tap, Decap) thường bị underestimate bởi người học mới — chúng không có logic function nhưng là bắt buộc về reliability: thiếu Tap Cells → latch-up risk; thiếu Filler Cells → broken N-Well implant → functional failure.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Views: [[LEF]] (physical) · [[LIB]] (timing) · [[GDS]] (layout) · Verilog (functional) · SPICE (transistor)
→ Complies with: [[Site]] (height/width) · [[RoutingGrid]] (Pin alignment)
→ Variants: Drive Strength · Multi-Vth (HVT/RVT/LVT) · Physical Cells (Filler/Tap/Decap/EndCap/ECO)
→ Consumed by: [[LogicSynthesis]] · [[GateLevelNetlist]] · [[Placement]] · [[Routing]] · [[ClockTreeSynthesis]] · [[STA]]
→ Cùng nhóm: [[HardIP]] · [[CellAbstract]] · [[Site]] · [[LEF]] · [[LIB]]
