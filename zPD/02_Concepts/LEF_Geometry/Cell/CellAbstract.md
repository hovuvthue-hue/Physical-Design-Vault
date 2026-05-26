---
tags: [concept, lef-geometry-cell]
group: LEF Geometry — Cell
defined_in: Macro LEF — generated từ GDS bởi Abstract Generator tool
used_by: [Placement, Routing, Floorplanning, DesignImport, ParasiticExtraction]
requires: [LEF, GDS, Site, Pin, Obstruction]
chain: Chain_LEF_to_PnR
---
# CellAbstract

## Definition
Cell Abstract (Physical Abstract View) là biểu diễn rút gọn của layout một Standard Cell hoặc Hard IP Macro trong Macro LEF, dùng cho Place & Route tool. Cell Abstract loại bỏ hoàn toàn tất cả layers bán dẫn bên dưới (transistor-level: Poly, Diffusion, N-Well, Contacts) và chỉ giữ lại đúng ba loại thông tin mà PnR tool cần: kích thước ngoài (PR Boundary), vị trí các Pins, và các vùng cấm routing (Obstructions).

Nhờ abstraction này, một SRAM macro chứa hàng triệu transistors với GDS hàng GB được rút gọn thành vài KB LEF text — PnR tool load và xử lý toàn bộ design trong RAM mà không bị overload.

Phạm vi cell types cần có CellAbstract trong PnR flow:
- Standard Cells (leaf cells): tất cả các logic functions từ Standard Cell library
- Memories: SRAM, ROM — generated bởi Memory Compiler
- Physical-Only Cells: Filler, Decap, Tie, End Cap — không có logic nhưng cần physical representation
- Hard IPs: PLL, PHY, ADC/DAC — IP Download hoặc Abstract Generation
- IO Pad cells: signal và power IO cells quanh chip periphery

Format chuẩn: **LEF**. Các format thay thế: **OA** (OpenAccess database) và **Milkyway** (Synopsys IC Compiler). Sự tồn tại của các format thay thế phản ánh lịch sử phân mảnh của EDA industry trước khi LEF trở thành standard de facto.

## Computed from
Cell Abstract được generate từ full layout (GDS) qua Abstract Generator (Cadence Abstract Generator hoặc Synopsys IC Compiler abstract generation). Quá trình gồm ba bước extract:

**Bước 1 — PR Boundary extraction**: Đọc cell boundary từ GDS → xác định `SIZE width BY height`. SIZE phải là bội số của Site dimensions để guarantee alignment với Placement Grid.

**Bước 2 — Pin extraction**: Identify các metal shapes trên signal/power layers mà designer đánh dấu là accessible → extract layer và coordinates của từng Pin. Pin phải nằm trên metal layer và align với Routing Grid Track positions.

**Bước 3 — Obstruction generation**: Tất cả metal shapes còn lại (internal routing của cell) được convert thành OBS — vùng tuyệt đối cấm external routing. Không có OBS, Router có thể vô tình đi dây xuyên qua internal metal của cell gây short circuits.

Cấu trúc Macro LEF điển hình của một Standard Cell (minh họa quan hệ giữa các thành phần):

MACRO NAND2X1
CLASS CORE ;
SIZE 0.560 BY 0.900 ;        ← PR Boundary
SITE core ;                   ← khai báo Site compliance
PIN A
DIRECTION INPUT ;
PORT LAYER M1 ; RECT 0.095 0.320 0.155 0.555 ; END
END A
PIN Y
DIRECTION OUTPUT ;
PORT LAYER M1 ; RECT 0.485 0.125 0.535 0.755 ; END
END Y
PIN VDD
USE POWER ; SHAPE ABUTMENT ;  ← power rail shared với adjacent cells
PORT LAYER M1 ; RECT 0.000 0.825 0.280 0.975 ; END
END VDD
OBS
LAYER CO ; RECT ... ;         ← contact layer internal → forbidden
LAYER M1 ; RECT ... ;         ← internal M1 routing → forbidden
END
END NAND2X1

## Constrains
- **[[Placement]]**: SIZE trong Cell Abstract xác định diện tích cell trên Placement Grid; Placement tool pack cells vào Rows mà không overlap dựa hoàn toàn vào SIZE — không cần biết transistors bên trong
- **[[Routing]]**: Pin locations là điểm kết nối duy nhất Router có thể access; OBS là vùng tuyệt đối cấm; Router phải route đến đúng Pin coordinates trên đúng metal layer
- **[[Floorplanning]]**: Hard IP Cell Abstracts (SRAM, PHY) với OBS lớn ảnh hưởng đến routing resource availability quanh Macro — phải account for OBS extent khi plan Macro locations để không tạo routing bottlenecks

## Requires
- [[LEF]] — Cell Abstract là nội dung chính của Macro LEF; format và syntax được define bởi LEF specification
- [[GDS]] — source data để generate Cell Abstract; Abstract Generator đọc GDS để extract PR Boundary, Pin locations, và internal metal shapes cho OBS
- [[Site]] — SIZE phải là multiple của Site dimensions; SITE type phải match library definition
- [[Pin]] — Pin locations phải align với Routing Grid Tracks
- [[Obstruction]] — OBS section định nghĩa vùng cấm để protect internal cell routing

## Used by
- [[DesignImport]] — toàn bộ Cell Abstracts (SC + Macro + Pad) phải được load trước khi init_design
- [[Placement]] — đọc SIZE và SITE để place cells trong Rows; tất cả area estimation và legalization dựa trên Cell Abstract SIZE
- [[Routing]] — đọc Pin locations để biết routing targets; đọc OBS để avoid; net-to-pin mapping đến từ Netlist, nhưng physical coordinates của Pins đến từ Cell Abstract
- [[Floorplanning]] — đọc Cell Abstract của Hard IP Macros để plan Macro placement và sizing routing channels

## Key insight
[USER REVIEW — draft suggestion]: Cell Abstract là implementation của nguyên lý "information hiding" trong Physical Design — mỗi tool chỉ nhận đúng thông tin nó cần. Circuit simulator cần transistor-level details → SPICE. Functional simulator cần logic behavior → Verilog. Layout verification cần full geometry → GDS. PnR tool cần placement và routing info → Cell Abstract (LEF). Sự phân tách này không chỉ là engineering elegance mà là necessity: nếu PnR tool phải load full GDS của tất cả cells, RAM của server sẽ cạn kiệt trong vài phút. Cell Abstract cho phép PnR tool của một billion-gate design fit trong vài GB RAM thay vì vài TB. Chất lượng Abstract Generator output ảnh hưởng trực tiếp đến routability — OBS quá conservative block routing channels không cần thiết; OBS thiếu gây DRC shorts sau Routing.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Format: Macro LEF — phần của [[LEF]] file; alternatives: OA · Milkyway
→ Generated from: [[GDS]] bởi Abstract Generator
→ Contains: PR Boundary (SIZE) · [[Pin]] locations · [[Obstruction]] regions
→ Must satisfy: SIZE = multiple of [[Site]] · Pins aligned with [[RoutingGrid]] Tracks
→ Consumed by: [[DesignImport]] · [[Placement]] · [[Routing]] · [[Floorplanning]]
→ Cùng nhóm: [[Pin]] · [[Obstruction]] · [[Site]] · [[Row]] · [[PlacementGrid]]