---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Chip-level .io file + Block-level TCL scripts; loaded during Design Initialization
used_by: [DesignImport, Floorplanning]
requires: [GateLevelNetlist, CellAbstract]
chain: Chain_PnR_Flow
---
# PhysicalConstraints

## Definition
Physical Constraints là tập hợp thông tin vật lý bắt buộc cung cấp cho PnR tool trước khi Floorplanning. Chúng capture floorplan intent ở mức khung: boundary, die/core size, IO pad/pin locations, macro-related constraints, và blockage intent. Có hai cấp độ tương ứng với hai loại design:

**Chip-level** (full-chip / top-level design):
- Die X/Y dimensions: kích thước tổng thể của chip — xác định bởi packaging constraints và system architect, không phải PD engineer
- IO Pad placement: vị trí các IO Cells trên 4 cạnh (top/left/bottom/right) — được chỉ định trong file `.io`

**Block-level** (hierarchical block trong SoC):
- PR boundary: ranh giới Place-and-Route của block — có thể là hình chữ nhật hoặc rectilinear (L-shape, T-shape) tùy theo chip topology
- Pin locations: vị trí các Ports của block trên PR boundary — là hard physical interface được inherit từ top-level integration và phải được honor

## Computed from

**Cấu trúc vật lý của chip từ ngoài vào trong:**

| Layer | Mô tả | Vai trò |
|---|---|---|
| Scribe Line | Vùng ngoài cùng | Cưa wafer khi dicing |
| Corner Cell | IO cell đặt tại 4 góc | Đảm bảo Ring liên tục tại corners |
| IO Cell (IO Pad) | Signal/power IO cells dọc 4 cạnh | Interface chip với package |
| Bond Pad / Bonding Area | Metal area lớn trên IO Cell | Wire-bond hoặc flip-chip bump attachment |
| IO Power Ring | Power ring ngoài cùng | Cấp nguồn cho IO cells |
| Core Power Ring | Power ring trong | Cấp nguồn cho Core logic |
| Core Area | Vùng logic chính | Chứa Standard Cells + Macro placements |

**File `.io` (Chip-level)** chứa instance names của các IO Pad cells được sorted theo vị trí trên 4 cạnh:

(iopad
(top    (inst name="pad_mdc"      place_status=placed) ...)
(left   (inst name="pad_dmack_n"  place_status=placed) ...)
(bottom (inst name="pad_em_d1_2"  place_status=placed) ...)
(right  (inst name="pad_em_a_10"  place_status=placed) ...)

Instance names trong `.io` phải match names trong GateLevelNetlist — mismatch gây error tại Design Initialization.

**Block-level Physical Constraints** (TCL scripts): khai báo PR boundary dimensions và Pin locations trên các cạnh. PR boundary xác định vùng mà PnR tool được phép place và route; ngoài boundary là forbidden zone. Rectilinear boundaries phổ biến khi block phải coexist với analog IP hoặc Macro có irregular shapes.

## Constrains
- **[[Floorplanning]]**: Die size và IO Pad positions là fixed starting point của Floorplan; không thể Floorplan nếu Physical Constraints chưa được load
- **[[Placement]]**: boundary, macro-related constraints và blockage intent giới hạn vùng place hợp lệ cho Standard Cells/Macros
- **[[Routing]]**: Block-level Pin locations phải align với RoutingGrid để Router kết nối block Ports với top-level Nets; misaligned Pins → DRC violations tại boundary
- **[[ClockTreeSynthesis]]**: trong một số designs, vị trí pin/boundary và macro-related constraints có thể ảnh hưởng topology phân phối clock [Needs verification]

## Requires
- [[GateLevelNetlist]] — instance names trong `.io` file phải match instance names trong Netlist
- [[CellAbstract]] — IO Pad cells cần LEF abstract để tool biết kích thước và Pin positions

## Used by
- [[DesignImport]] — Physical Constraints là input thứ 4, cùng với Design Initialization
- [[Floorplanning]] — die dimensions và IO Pad positions là điểm khởi đầu bắt buộc

## Key insight
[USER REVIEW — draft suggestion]: Die X/Y dimensions không được PD engineer tự chọn — chúng bị ràng buộc chặt bởi packaging constraints (wire bond pitch, flip-chip bump grid, package size) từ Package team và area/cost targets từ Management. PD engineer nhận die size như một input; nhiệm vụ là fit design vào die size đó với utilization hợp lý (thường 60–80%). Block-level Pin locations là điểm đàm phán quan trọng: nếu Pin của block A không align với Port của block B ở top-level, Router phải thêm detour routing hoặc buffer — cả hai đều gây congestion và timing degradation. Pin assignment meeting giữa block teams là một step bắt buộc trong hierarchical design flow.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Chip-level: Die dimensions + IO Pad placement (.io file)
→ Block-level: PR boundary + Pin locations (TCL scripts)
→ Chip anatomy: Scribe Line → Corner Cell → IO Cell → Bond Pad → IO Power Ring → Core Power Ring → Core Area
→ Consumed by: [[DesignImport]] · [[Floorplanning]] · [[Routing]]
→ Cùng nhóm: [[Floorplanning]] · [[GateLevelNetlist]] · [[CellAbstract]]