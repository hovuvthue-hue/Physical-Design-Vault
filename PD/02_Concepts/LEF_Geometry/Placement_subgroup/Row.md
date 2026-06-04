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

Đọc là: ROW_1 sử dụng Site type "core", bắt đầu tại (0,0), chứa các Sites theo X với step bằng Site width. 

Row orientation xen kẽ giữa hai giá trị chuẩn để enable cell flipping:

| Orientation | Ý nghĩa |
|---|---|
| **R0** | Upright — không rotation, không mirror |
| **MY** | Flip top to bottom — mirror theo trục ngang (X-axis) |

> **Lưu ý convention:** Tresemi L7 gọi row orientation "flip top to bottom" là **MY**, trong khi convention EDA chuẩn thường dùng **MX** (Mirror about X-axis = horizontal axis) cho flip top-to-bottom và **MY** cho flip left-to-right. L7 slide p.9 định nghĩa "MY – Mirror along Y-axis" nhưng p.10 lại mô tả "MY: Flip top to bottom" — hai phát biểu này không nhất quán với nhau. Khi làm việc với Innovus hoặc ICC2, cần verify orientation naming theo tool documentation của từng phiên bản. **Điều không thay đổi:** physical effect cần thiết cho PG rail abutment là flip top-to-bottom giữa các hàng liền kề.

Pattern R0 → MY → R0 → MY giữa các hàng liền kề tạo ra hai hiệu ứng vật lý bắt buộc:
- **Abutted PG Rail**: VDD/VSS rail ở biên trên hàng R0 tiếp giáp rail ở biên dưới hàng MY liền kề — cells chia sẻ PG rail tại biên row mà không cần routing bổ sung.
- **Abutted N-Well**: N-Well của cells trong hai hàng liền kề tiếp giáp nhau — loại bỏ well-to-well gap giữa các hàng, giảm DRC spacing requirements.

Đây là cơ chế nền tảng cho phép Standard Cells được abutted trong Rows mà vẫn đảm bảo power/well connectivity liên tục.

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
Row orientation xen kẽ R0 (upright) và MY (flip top to bottom). Cơ chế này tạo ra Abutted PG Rail và Abutted N-Well tại biên giữa các hàng liền kề — đây là điều kiện để Standard Cell fabric có power delivery và well biasing liên tục mà không cần khoảng cách giữa rows hay routing bổ sung. Library designer thiết kế cells với VDD rail ở một cạnh và VSS rail ở cạnh kia chính xác để tương thích với pattern abutment này. Macro placement không tương thích với Row/Site boundary có thể làm giảm vùng placement hợp lệ quanh macro hoặc tạo local placement/routing difficulty.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formed from: [[Site]] tiles (horizontal direction)
→ Stored in: [[DEF]] — ROWS section post-Floorplan
→ Generated during: [[Floorplanning]]
→ Aggregated into: [[PlacementGrid]]
→ Contains: [[Placement]] locations for Standard Cells
→ Orientation: alternating row orientation, naming phụ thuộc library/tool convention [Needs verification]
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Site]] · [[PlacementGrid]]
