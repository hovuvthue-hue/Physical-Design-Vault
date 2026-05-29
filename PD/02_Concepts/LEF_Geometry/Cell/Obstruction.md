---
tags: [concept, lef-geometry]
group: LEF Geometry — Cell
defined_in: Macro LEF — OBS section trong CellAbstract
used_by: [Routing, Placement, Floorplanning]
requires: [CellAbstract, LEF]
chain: Chain_LEF_to_PnR
---
# Obstruction

## Definition
Obstruction (OBS) là vùng hình học trong Cell Abstract (Macro LEF) đánh dấu các khu vực trên specific metal layers mà Router tuyệt đối không được phép đặt wire của external nets đi qua. OBS phát sinh từ internal metal routing của cell — các wire segments kết nối transistors bên trong cell đã chiếm dụng một số Track positions trên M1, M2 và các layers khác. Nếu Router không biết về các vùng này, nó có thể route external nets đi qua bên trong cell, tạo ra shorts với internal wires — đây là DRC violation nghiêm trọng nhất (short circuit).

## Computed from
OBS được Abstract Generator extract từ GDS bằng cách identify tất cả metal shapes bên trong cell boundary không phải là Pin:

OBS
LAYER CO ;
RECT 0.190 0.130 0.230 0.170 ;
RECT 0.190 0.715 0.230 0.755 ;
LAYER M1 ;
RECT 0.200 0.200 0.400 0.600 ;
END

OBS có thể cover toàn bộ cell area trên một layer (full OBS — phổ biến ở Hard IP Macros như SRAM) hoặc chỉ cover một số vùng cụ thể (partial OBS — phổ biến ở Standard Cells).

**Hard IP OBS pattern**: SRAM macro thường có OBS cover toàn bộ M1 đến M4 trên toàn bộ footprint — vì tất cả internal routing đã được done bởi memory compiler. External connections chỉ được phép ở M5 trở lên (layers trên OBS extent). PD engineer phải account cho điều này khi plan routing channels quanh SRAM.

**Standard Cell OBS pattern**: Standard Cells thường có partial OBS — chỉ những vùng internal metal thực sự chiếm dụng mới được mark là OBS; phần còn lại của cell area (trên các layers cao hơn như M2, M3) vẫn có thể được Router dùng để route over-cell wires (feedthrough routing).

Placement Blockage (khác với Routing OBS): một khái niệm riêng biệt trong Floorplanning — vùng cấm Placement (không phải Routing). PD engineer có thể define Placement Blockages thủ công trong Floorplanning để reserve vùng cho specific purposes (ví dụ: vùng gần analog blocks không được phép place digital cells vì noise coupling).

## Constrains
- **[[Routing]]**: OBS là hard constraint — Router phải route tất cả external nets around OBS regions trên các layers bị blocked; không có exceptions; việc Router bypass OBS constraints sẽ tạo DRC short violations không thể fix mà không re-route
- **[[Floorplanning]]**: Hard IP OBS extent ảnh hưởng đến routing channel planning — vùng quanh SRAM macro với M1–M4 OBS chỉ còn M5+ cho external routing, làm routing capacity giảm đáng kể; PD engineer phải plan đủ routing channels (routing margin) quanh Macros

## Requires
- [[CellAbstract]] — OBS là section bên trong Cell Abstract (Macro LEF); OBS không tồn tại độc lập mà luôn là thuộc tính của một specific cell
- [[LEF]] — LEF format định nghĩa OBS syntax; OBS được read bởi PnR tool khi loading Cell Abstract

## Used by
- [[Routing]] — Router reads OBS từ tất cả Cell Abstracts trước khi bắt đầu routing; build internal "blockage map" từ OBS data; routing algorithm avoid blocked Track segments; Hard IP OBS với extent nhiều layers làm giảm routing capacity đáng kể trong vùng lân cận
- [[Placement]] — một số Placement tools dùng OBS information để estimate routing difficulty và avoid placing nhiều cells với large OBS trong cùng một vùng (congestion-aware placement)
- [[Floorplanning]] — PD engineer phải analyze OBS extent của Hard IP Macros khi planning Macro placement; Macros với large OBS (ví dụ SRAM M1–M6 OBS) cần được placed ở vùng có đủ routing layers phía trên (M7+) để allow over-the-macro routing

## Key insight
[USER REVIEW — draft suggestion]: OBS là safety mechanism ngăn chặn một class of catastrophic errors trong automated routing — nếu không có OBS, Router sẽ hành xử như "không biết" cell có internal metals và vô tư route qua bên trong, tạo ra shorts chỉ có thể được phát hiện bởi LVS sau Routing (quá muộn và tốn kém để fix). OBS là concrete example của tại sao "information hiding" trong abstraction không phải là tùy chọn — đây là safety-critical information. Điểm thực tế quan trọng: khi integrate một Hard IP từ external vendor, chất lượng OBS trong Cell Abstract là critical — OBS quá conservative (block nhiều hơn cần thiết) sẽ tạo routing difficulty không cần thiết; OBS không đủ (miss một số internal metals) sẽ gây shorts. Đây là lý do IP qualification process luôn bao gồm bước verify Cell Abstract accuracy trước khi use trong design.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Defined in: [[CellAbstract]] — Macro LEF OBS section
→ Generated from: internal metal shapes trong [[GDS]]
→ Hard constraint for: [[Routing]] — no external wires allowed in OBS regions
→ Affects: routing capacity analysis in [[Floorplanning]]
→ Types: Full OBS (Hard IP) · Partial OBS (Standard Cell)
→ Distinguished from: Placement Blockage (different concept — forbids cell placement, not routing)
→ Cùng nhóm: [[CellAbstract]] · [[Pin]] · [[Site]] · [[Row]] · [[RoutingGrid]]