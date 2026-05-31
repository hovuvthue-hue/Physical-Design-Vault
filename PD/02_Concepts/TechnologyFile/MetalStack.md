---
tags: [concept, lef-geometry, technology]
group: Technology Files
defined_in: Tech LEF (layer definitions) + ITF (RC parameters) — provided by Foundry
used_by: [Routing, ParasiticExtraction, Floorplanning, Signoff]
requires: [LEF, ITF]
chain: Chain_LEF_to_PnR
---
# MetalStack

## Definition
Metal Stack là toàn bộ hệ thống các lớp kim loại dẫn điện (metal layers) xếp chồng lên nhau theo chiều thẳng đứng trong một IC, được ngăn cách bởi các lớp điện môi (dielectric), và được kết nối với nhau qua các [[Via]]. Metal Stack là không gian vật lý ba chiều mà toàn bộ signal routing, power distribution, và clock delivery xảy ra. Foundry định nghĩa Metal Stack cho từng process node — số lượng layers, thickness, resistivity, và dielectric properties của mỗi layer là fixed constraints mà PD engineer không thể thay đổi.

## Computed from
Metal Stack được Foundry phát triển trong quá trình technology node development và cung cấp cho khách hàng qua Tech LEF và ITF files. Cấu trúc ví dụ của một Metal Stack trong slide [Needs verification]:

| Layer | Type            | Purpose                          | Đặc điểm                       |
| ----- | --------------- | -------------------------------- | ------------------------------ |
| M1    | Local (x)       | Routing bên trong Standard Cells | Nhỏ nhất, mật độ cao nhất      |
| M2–M6 | Standard (x)    | Inter-cell signal routing        | Balanced performance & density |
| M7    | Thick (z)       | Clock nets, wide signals         | Resistance thấp, EM-safe       |
| M8    | Ultra-thick (u) | Power grid, global signals       | High current capacity          |
| AP    | Aluminum Pad    | I/O bonding pads                 | Không dùng cho routing         |

TSMC naming convention cho Metal Stack [Needs verification]:
- **x** (standard-thickness): dense signal routing
- **y** (thicker): semi-global routing, lower resistance
- **z** (even-thicker): global signals và clock distribution
- **u** (ultra-thick): power distribution — minimal resistance
- **r** (redistribution): I/O connections và bumping

Lưu ý quan trọng: M1 (Local routing layer) luôn có mặt trong mọi stack
nhưng được OMIT khỏi tên — tên stack chỉ khai báo M2 trở lên.

Ví dụ: "5x2y2z" thực ra gồm M1 + 5×Mx + 2×My + 2×Mz = 10 metal layers.
Cách đọc: total_count = 1 (M1) + sum(layer_counts_in_name).

Cách diễn giải tên stack trong ví dụ phụ thuộc convention của foundry/process cụ thể [Needs verification].

Trade-off trong Metal Stack selection:
- **High-Performance stack** (nhiều thick layers): lower resistance → faster signals → better timing, nhưng fewer layers → lower routing density
- **High-Density stack** (nhiều thin layers): more layers → higher routing density → smaller chip, nhưng higher resistance → worse timing và more IR drop

## Constrains
- **[[Routing]]**: Metal Stack xác định routing resources available cho toàn bộ design — số layers, Pitch per layer, preferred routing direction per layer; PD engineer phải plan layer usage strategy (layer assignment) để balance signal routing vs power routing
- **[[ParasiticExtraction]]**: thickness, resistivity, và dielectric constants của từng layer trong Metal Stack là physical parameters cho RC extraction; layer stackup trong ITF là input trực tiếp để tính [[InterconnectRC]] từ routed metal/via geometry
- **[[Floorplanning]]**: PDN design phải chọn đúng layers cho power stripes — Ultra-thick layers (u-type) cho global power rails (thấp nhất resistance = ít IR drop nhất); Standard layers cho local power connections

## Requires
- [[LEF]] — Tech LEF chứa LAYER definitions cho tất cả metal layers trong Metal Stack: PITCH, DIRECTION, MINWIDTH, SPACING, RESISTANCE (per unit length), CAPACITANCE
- [[ITF]] — Interconnect Technology File chứa physical parameters của Metal Stack cho RC extraction, giúp tính [[InterconnectRC]] theo layer và corner

## Used by
- [[Routing]] — Router assign nets đến specific layers trong Metal Stack dựa trên net criticality và layer properties; clock nets được assign to thick layers (lower RC → less clock delay); signal nets distributed across standard layers
- [[ParasiticExtraction]] — extraction tool reads ITF Metal Stack parameters để compute $R = R_\square \times L/W$ và $C = \varepsilon \times A / t_{di}$ cho từng wire segment; accuracy của SPEF phụ thuộc vào accuracy của Metal Stack parameters trong ITF
- [[Floorplanning]] — PDN planning phải select appropriate Metal Stack layers: ultra-thick layers cho power mesh (minimum IR drop), thick layers cho clock trunk routing (minimum insertion delay)
- [[Signoff]] — Metal density DRC rules kiểm tra từng layer trong Metal Stack có đủ metal coverage (để maintain uniform etch rate trong fab process); EM (Electromigration) rules verify current density trong each layer không exceed safe limit

## Key insight
[USER REVIEW — draft suggestion]: Metal Stack là lý do tại sao advanced node chips tốn hàng tỷ USD để phát triển — mỗi layer đại diện cho một mask trong fab process, và ở một số process/foundry cụ thể số layer và mask cost có thể rất lớn [Needs verification]. Từ góc độ Physical Design, Metal Stack quyết định "routing budget" — số Track resources available trên toàn chip. Một insight quan trọng: khi chip cần thêm routing resources và không thể increase die size, giải pháp là thêm metal layers — đây là một trong những lý do advanced node chips thường có nhiều layers hơn các node cũ trong một số family/process cụ thể [Needs verification]. Trade-off là cost — mỗi layer thêm vào tăng mask cost và cycle time của fab.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Defined in: [[LEF]] (LAYER section) + [[ITF]] (RC parameters)
→ Provided by: Foundry (TSMC, Samsung, Intel Foundry...)
→ Layer types: Local (x) · Standard (x) · Thick (z) · Ultra-thick (u) · Redistribution (r)
→ Affects: [[Routing]] layer assignment · [[Via]] usage · [[InterconnectRC]] · [[ParasiticExtraction]] RC values · PDN in [[Floorplanning]]
→ Scaling trend: more layers per node → higher routing density but higher mask cost
→ Cùng nhóm: [[ITF]] · [[LEF]] · [[GDS]]