---
tags: [concept, file-format, lef-geometry]
group: File Formats
defined_in: Foundry (Tech LEF) · Library vendor / IP provider (Macro LEF) — ASCII text format
used_by: [Floorplanning, Placement, Routing, ParasiticExtraction, Signoff]
requires: [N/A — LEF là input gốc từ Foundry/Library, không được generate từ EDA flow]
chain: Chain_LEF_to_PnR
---
# LEF

## Definition
LEF (Library Exchange Format) là file format ASCII text định nghĩa thông tin vật lý của technology và Standard Cell library dùng cho Place & Route tool. LEF thực hiện abstraction — ẩn đi toàn bộ chi tiết bán dẫn bên trong cell (transistor, diffusion, poly) và chỉ giữ lại đúng những gì PnR tool cần: kích thước ngoài, vị trí Pins, và vùng cấm routing. Có hai loại LEF hoàn toàn khác nhau về vai trò: Tech LEF (do Foundry cung cấp — định nghĩa technology rules) và Macro LEF (do library vendor cung cấp — định nghĩa từng cell cụ thể).

## Computed from
LEF không được compute trong EDA flow mà được cung cấp bởi Foundry và library vendor như một input artifact. Tuy nhiên, Macro LEF được generate từ full layout (GDS) thông qua một công cụ gọi là Abstract Generator — tool này đọc GDS, strip bỏ tất cả các layers không liên quan đến PnR, và output LEF chỉ chứa metal layers.

**Tech LEF** (P&R Tech File) chứa các sections chính: routing layers, DRC rules, Site, Track definitions
- `UNITS`: đơn vị đo lường (micron, pF, ohm)
- `MANUFACTURINGGRID`: grid tối thiểu của Foundry
- `SITE`: định nghĩa kích thước tối thiểu của Standard Cell (width × height)
- `LAYER (ROUTING)`: mỗi metal layer có DIRECTION (H/V), PITCH, MINWIDTH, SPACING, RESISTANCE, CAPACITANCE
- `LAYER (CUT/VIA)`: via layers với SPACING, ENCLOSURE rules
- `VIA`: via definitions với geometry
- `PROPERTYDEFINITIONS`: design rule strings

**Macro LEF** (Cell Abstracts) chứa cho từng cell: SC.lef, Macro.lef, Pad.lef — physical representation của cells
- `MACRO`: tên cell, SIZE (width × height), SYMMETRY, SITE
- `PIN`: tên pin, DIRECTION (INPUT/OUTPUT), USE (SIGNAL/POWER), PORT với LAYER và RECT coordinates
- `OBS`: Obstruction — vùng cấm routing với LAYER và RECT coordinates

## Constrains
- **[[Placement]]**: SITE definition trong Tech LEF xác định Placement Grid — mọi Standard Cell phải được đặt align với SITE boundaries; cell width phải là bội số của SITE width; cell height bằng đúng SITE height
- **[[Routing]]**: LAYER definitions trong Tech LEF xác định Routing Grid (PITCH, DIRECTION per layer) — Routing tool chỉ được route trên các tracks được define; OBS trong Macro LEF là vùng cấm tuyệt đối
- **[[ParasiticExtraction]]**: Tech LEF cung cấp layer stackup parameters (THICKNESS, RESISTANCE, CAPACITANCE per layer) — là input cho RC extraction model; extraction tool đọc LEF để biết vật liệu và geometry của mỗi metal layer
- **[[Signoff]]**: DRC rules trong Tech LEF (MINWIDTH, SPACING, ENCLOSURE) là reference cho physical verification; LVS dùng Macro LEF để verify connectivity

## Requires
N/A — LEF là primary input artifact, không phụ thuộc vào concept kỹ thuật upstream trong EDA flow. Tech LEF được Foundry phát triển dựa trên process node characteristics; Macro LEF được library vendor generate từ GDS layout.

## Used by
- [[Floorplanning]] — đọc SITE từ Tech LEF để define Row structure và Placement Grid; đọc Macro LEF của Hard IPs (SRAM, analog blocks) để biết kích thước và vị trí Pins khi plan floorplan
- [[Placement]] — đọc Macro LEF của tất cả Standard Cells để biết SIZE và Pin locations; align cell placement theo SITE grid; respect OBS của các cells đã placed
- [[Routing]] — đọc Tech LEF để biết PITCH/DIRECTION per layer → xây dựng Routing Grid; đọc OBS trong Macro LEF để avoid routing qua nội bộ cells; check SPACING rules của LAYER definitions
- [[ParasiticExtraction]] — đọc Tech LEF layer properties để compute R và C per unit length; kết hợp với wire geometry từ DEF để extract SPEF
- [[Signoff]] — DRC tool đọc Tech LEF design rules; LVS tool đọc Macro LEF connectivity

## Key insight
[USER REVIEW — draft suggestion]: LEF là embodiment của nguyên lý Abstraction trong VLSI — nó giải quyết bài toán scale bằng cách ẩn đi complexity không cần thiết. PnR tool không cần biết bên trong một SRAM macro có bao nhiêu transistor hay các bitcells được layout như thế nào — nó chỉ cần biết "khối này rộng 500um, cao 500um, chân CLK nằm ở tọa độ (10, 250) trên Metal 4, và toàn bộ bên trong là OBS từ M1 đến M3." Đây là lý do LEF làm cho chip integration scalable: thay vì load hàng GB GDS data của một SRAM vào bộ nhớ, PnR tool chỉ cần vài KB LEF text. Điểm thực tế: khi một Hard IP vendor giao IP, họ không bao giờ giao GDS để chạy PnR — họ giao LEF (physical) + LIB (timing) + Verilog (functional), đây là bộ 3 tối thiểu để integrate một Hard IP.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Hai loại: Tech LEF (Foundry) · Macro LEF (Library vendor / IP provider)
→ Generated from: [[GDS]] (qua Abstract Generator tool)
→ Defines: [[Site]] · [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Row]] · [[PlacementGrid]] · [[CellAbstract]] · [[Obstruction]]
→ Used alongside: [[LIB]] (timing) · [[GDS]] (full layout) · [[DEF]] (design instance data)
→ Consumed by: [[Floorplanning]] · [[Placement]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]]
→ Cùng nhóm: [[LIB]] · [[SPEF]] · [[DEF]] · [[GDS]]