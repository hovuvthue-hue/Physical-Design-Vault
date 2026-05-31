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
Row là một dải nằm ngang (horizontal strip) trong Standard Cell placement area, được formed bằng cách tile Site units liền kề theo chiều ngang, tạo thành hạ tầng legal placement lặp lại cho Standard Cells. Row height = Site height; Row width = (số Sites) × Site width. Standard Cells chỉ được placed trong Rows — không thể placed ở vùng không có Row (ví dụ: bên trong Macro, vùng blockage, hoặc ngoài Core area). Rows liền kề thường được tổ chức theo pattern đối xứng/đảo hướng để hỗ trợ tính liên tục của power rails và well continuity ở mức hạ tầng. Cách đặt orientation cụ thể là tool/library-dependent [Needs verification].

## Computed from
Rows được generate trong Floorplanning step dựa trên Core area dimensions và Site definition, sau đó stored trong DEF:

ROW ROW_1 core 0 0 N DO 500 BY 1 STEP 140 0 ; 
ROW ROW_2 core 0 900 FS DO 500 BY 1 STEP 140 0 ;

Đọc là: ROW_1 sử dụng Site type "core", bắt đầu tại (0,0), chứa các Sites theo X với step bằng Site width. ROW liền kề có thể dùng orientation đối xứng/đảo hướng để phù hợp rail/well topology của library [Needs verification].

$$\text{Tổng số Rows} = \left\lfloor \frac{\text{Core Height}}{\text{Site Height}} \right\rfloor$$

$$\text{Sites per Row} = \left\lfloor \frac{\text{Core Width}}{\text{Site Width}} \right\rfloor$$

Vùng bên trong Macro footprint không có Rows (Macro chiếm diện tích và không cho phép Standard Cell placement). PD engineer phải account for Macro placements khi counting available Rows cho Standard Cell placement.

## Constrains
- **[[PlacementGrid]]**: tất cả Rows trên Core area tạo thành Placement Grid 2D; mỗi Site position trong mỗi Row là một valid placement location
- **[[Placement]]**: Standard Cells chỉ được placed trong Rows; cells với multi-height có thể cần span nhiều Rows liên tiếp với orientation tương thích theo library/tool rule [Needs verification]
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
Row orientation thường được tổ chức theo pattern xen kẽ hoặc đối xứng để hỗ trợ power-rail/well continuity của standard-cell architecture; tên orientation cụ thể và legality rule phụ thuộc library/tool convention. [Needs verification] Macro placement không tương thích với Row/Site boundary có thể làm giảm vùng placement hợp lệ quanh macro hoặc tạo local placement/routing difficulty.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formed from: [[Site]] tiles (horizontal direction)
→ Stored in: [[DEF]] — ROWS section post-Floorplan
→ Generated during: [[Floorplanning]]
→ Aggregated into: [[PlacementGrid]]
→ Contains: [[Placement]] locations for Standard Cells
→ Orientation: alternating row orientation, naming phụ thuộc library/tool convention [Needs verification]
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Site]] · [[PlacementGrid]]
