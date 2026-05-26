---
tags: [concept, lef-geometry]
group: LEF Geometry — Placement
defined_in: DEF — ROWS section (post-Floorplan); derived from Site + Core area
used_by: [PlacementGrid, Placement, Floorplanning]
requires: [Site, LEF, DEF]
chain: Chain_LEF_to_PnR
---
# Row

## Definition
Row là một dải nằm ngang (horizontal strip) trên Core area được formed bằng cách tile Site units liền kề theo chiều ngang, tạo thành một "hàng" mà Standard Cells được placed vào. Row height = Site height; Row width = (số Sites) × Site width. Standard Cells chỉ được placed trong Rows — không thể placed ở vùng không có Row (ví dụ: bên trong Macro, vùng blockage, hoặc ngoài Core area). Rows liền kề thường được đặt theo kiểu flip alternating để VDD/VSS rails của các Rows kề nhau chia sẻ chung một rail — đây là cơ chế giúp Power Distribution Network (PDN) được deliver đến mọi cell hiệu quả.

## Computed from
Rows được generate trong Floorplanning step dựa trên Core area dimensions và Site definition, sau đó stored trong DEF:

ROW ROW_1 core 0 0 N DO 500 BY 1 STEP 140 0 ; 
ROW ROW_2 core 0 900 FS DO 500 BY 1 STEP 140 0 ;

Đọc là: ROW_1 sử dụng Site type "core", bắt đầu tại (0,0), orientation N (normal), chứa 500 Sites theo X với step 140nm (= Site width), 1 Site theo Y. ROW_2 tại Y=900 (= 1 Site height) với orientation FS (flip + symmetric) — orientation đảo ngược để VDD của ROW_2 tiếp giáp với VDD của ROW_1, VSS tiếp giáp VSS.

$$\text{Tổng số Rows} = \left\lfloor \frac{\text{Core Height}}{\text{Site Height}} \right\rfloor$$

$$\text{Sites per Row} = \left\lfloor \frac{\text{Core Width}}{\text{Site Width}} \right\rfloor$$

Vùng bên trong Macro footprint không có Rows (Macro chiếm diện tích và không cho phép Standard Cell placement). PD engineer phải account for Macro placements khi counting available Rows cho Standard Cell placement.

## Constrains
- **[[PlacementGrid]]**: tất cả Rows trên Core area tạo thành Placement Grid 2D; mỗi Site position trong mỗi Row là một valid placement location
- **[[Placement]]**: Standard Cells chỉ được placed trong Rows; cells với double height phải span hai Rows liên tiếp với matching orientation; cell utilization = (total cell area) / (total Row area) — thường target 70–80%
- **[[Floorplanning]]**: Row generation là một output của Floorplanning; Macro placement affects Row availability — Macros phải placed tại Row boundaries (height = N × Site height) để không bisect một Row; đây là constraint quan trọng khi plan Macro locations

## Requires
- [[Site]] — Row được formed từ Sites; Row height = Site height; Row width = N × Site width; không có Site definition, không thể create Rows
- [[LEF]] — Site definition trong Tech LEF là prerequisite cho Row generation
- [[DEF]] — Rows được stored và managed trong DEF ROWS section; sau Floorplanning, DEF chứa tất cả Row definitions với exact coordinates và orientations

## Used by
- [[PlacementGrid]] — tập hợp tất cả Site positions trong tất cả Rows tạo thành Placement Grid; Row là horizontal slice của PlacementGrid
- [[Placement]] — Placement tool reads Rows từ DEF và places cells vào valid Site positions trong Rows; Legalization step ensures không có cells overlap và tất cả cells aligned với Row grid
- [[Floorplanning]] — Floorplanning tool generates Rows sau khi Core area và Macro positions được defined; Row generation là automated step nhưng PD engineer phải verify Row coverage và identify gaps do Macro placements

## Key insight
[USER REVIEW — draft suggestion]: Row orientation alternating (N/FS/N/FS...) là một trong những design patterns elegant nhất trong Physical Design — bằng cách flip alternating Rows, VDD rail của Row chẵn tiếp giáp với VDD rail của Row lẻ (và VSS tiếp giáp VSS), tạo ra continuous power rails chạy ngang chip mà không cần via connections giữa Rows. Đây vừa tiết kiệm via resistance vừa giúp PDN delivery uniform trên toàn chip. Điểm thực tế: khi Macro không được placed tại Row boundaries (Macro height không phải bội số của Site height), nó sẽ "bisect" một Row tạo ra incomplete Rows với một số Sites không thể dùng được — đây là DRC violation hoặc placement legalization error phổ biến khi integrate Hard IP macros không được designed cho một specific library. Đây là lý do Macro height luôn phải là bội số của Site height trong library specification.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formed from: [[Site]] tiles (horizontal direction)
→ Stored in: [[DEF]] — ROWS section post-Floorplan
→ Generated during: [[Floorplanning]]
→ Aggregated into: [[PlacementGrid]]
→ Contains: [[Placement]] locations for Standard Cells
→ Orientation: alternating N/FS để share VDD/VSS rails
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Site]] · [[PlacementGrid]]