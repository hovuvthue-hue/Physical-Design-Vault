---
tags: [concept, lef-geometry]
group: LEF Geometry — Cell
defined_in: Macro LEF — PIN section trong CellAbstract; cũng referenced trong GateLevelNetlist (pin names)
used_by: [Routing, CellAbstract, STA, Placement]
requires: [CellAbstract, LEF, Track, RoutingGrid]
chain: Chain_LEF_to_PnR
---
# Pin

## Definition
Pin là điểm kết nối vật lý của một cell — vùng metal trên một cụ thể metal layer tại đó Router có thể đặt một Via hoặc wire end để kết nối net vào cell đó. Pin trong context LEF/Physical Design khác với Pin trong context STA: STA Pin là logical concept (input/output của một timing arc), còn LEF Pin là physical concept (geometric shape tại specific coordinates trên specific metal layer). Một logical Pin từ Netlist được map đến physical Pin location từ Cell Abstract để Router biết chính xác cần đặt wire đến đâu.

## Computed from
Pin được định nghĩa trong Macro LEF với đầy đủ thông tin vật lý:

PIN A
DIRECTION INPUT ;
USE SIGNAL ;
ANTENNAGATEAREA 0.009300 ;
PORT
LAYER M1 ;
RECT 0.095 0.320 0.155 0.555 ;
END
END A

Các thuộc tính quan trọng:
- **DIRECTION**: INPUT / OUTPUT / INOUT / FEEDTHRU
- **USE**: SIGNAL / POWER / GROUND / CLOCK / ANALOG
- **LAYER + RECT**: physical location — layer nào, tọa độ (x1 y1 x2 y2) nào
- **ANTENNAGATEAREA**: diện tích gate oxide — dùng cho antenna rule checking (DRC)

**Pin Alignment constraint — quan trọng nhất:**
Tâm của Pin RECT phải nằm trúng tại giao điểm của Routing Grid (Track intersection). Cụ thể: tọa độ Y của Pin center phải là bội số của M1 Pitch (để M1 Track align), và tọa độ X phải là bội số của M2 Pitch (để M2 Track align). Nếu Pin off-grid, Router phải tạo jog wire để tiếp cận — tốn Track resources và có thể gây spacing DRC.

Phân biệt **Pin** và **Port** trong EDA:
- **Port**: điểm kết nối ở top-level boundary của design (Primary Input/Output) — dùng trong SDC (get_ports)
- **Pin**: điểm kết nối của một cell instance bên trong design — dùng trong SDC (get_pins) và LEF

## Constrains
- **[[Routing]]**: Pin locations là routing targets — Router phải đặt wire end hoặc Via tại Pin RECT; Pin layer xác định Router cần access từ layer nào; [[PinAccess]] (dễ hay khó tiếp cận pin hợp lệ) phụ thuộc vào alignment với Routing Grid
- **[[STA]]**: STA timing arc bắt đầu và kết thúc tại Pins; Physical Pin location ảnh hưởng đến wire length từ driver Pin đến receiver Pin → ảnh hưởng Net Delay; post-Placement, STA estimate Net Delay dựa trên Manhattan distance giữa driver Pin và receiver Pin coordinates
- **[[CellAbstract]]**: Pin là thành phần chính của Cell Abstract; không có Pin definitions, Router không biết cần connect đến đâu

## Requires
- [[CellAbstract]] — Pin được defined bên trong Macro LEF (Cell Abstract); Pin không tồn tại độc lập ngoài context của cell
- [[LEF]] — LEF format specification định nghĩa syntax của PIN section
- [[Track]] · [[RoutingGrid]] — Pin phải align với Track grid; alignment requirement là constraint từ Routing Grid structure

## Used by
- [[Routing]] — Router reads Pin locations từ Cell Abstract để build routing targets; Router places Via tại Pin location để connect net wire đến cell; [[PinAccess]] analysis là một routing-feasibility concern quanh pin
- [[CellAbstract]] — Pin là primary content của Cell Abstract cùng với OBS; Cell Abstract không có giá trị nếu không có accurate Pin definitions
- [[STA]] — STA tool map logical pin names (từ Netlist) đến physical pin coordinates (từ Cell Abstract) để compute wire length estimates; Timing Arc start/end tại logical Pins nhưng Net Delay computation cần physical Pin coordinates

## Key insight
[USER REVIEW — draft suggestion]: Pin alignment với Routing Grid là một trong những constraints phổ biến nhất gây lỗi khi porting một library sang một technology node mới. Nếu Pin center coordinates trong Cell Abstract không align với Track grid của Tech LEF mới, Router phải tạo jog wires cho mọi Pin access — multiplied across hàng triệu cells tạo ra massive routing overhead. Đây là lý do library development (tạo cell layouts) phải được done với tech-specific grid awareness, và tại sao Library Characterization và Technology Migration là projects tốn kém. Thực tế: một số advanced PD flows dùng "Pin access point analysis" như một standalone step sau Placement để pre-compute tất cả valid Via positions cho từng Pin trước khi Routing bắt đầu — giúp Router chạy faster và với better quality.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Defined in: [[CellAbstract]] — Macro LEF PIN section
→ Must align with: [[Track]] · [[RoutingGrid]] (Pin center on Track intersection)
→ Routing target for: [[Routing]] · [[PinAccess]]
→ Logical counterpart: logical pin names trong [[GateLevelNetlist]]
→ Timing endpoint for: [[STA]] — [[TimingArc]] start/end
→ Distinguished from: Port (top-level boundary connection)
→ Cùng nhóm: [[CellAbstract]] · [[Obstruction]] · [[Site]] · [[Track]] · [[RoutingGrid]]