---
tags: [concept, lef-geometry]
group: LEF Geometry — Placement
defined_in: Tech LEF — SITE statement (cùng file với TRACKS)
used_by: [Row, PlacementGrid, Placement, CellAbstract]
requires: [LEF, Pitch]
chain: Chain_LEF_to_PnR
---
# Site

## Definition
Site là đơn vị hình học hợp lệ nhỏ nhất (minimum legal placement unit) được định nghĩa trong Tech LEF, đóng vai trò legal anchor cho cell origin trong [[Placement]]. Cell width là bội số nguyên của Site width. Cell height là bội số nguyên của Site height; single-height standard cells thường có height bằng một Site height, còn multi-height cells dùng nhiều Site heights. Site đảm bảo tính regularity của placement theo site-grid alignment, để cells có thể xếp vào [[Row]] một cách hợp lệ và không overlap sau legalization.

## Computed from
Site được Foundry định nghĩa trong Tech LEF:

SITE core 
SIZE 0.140 BY 0.900 ; 
SYMMETRY Y ; 
CLASS CORE ; 
END core

Chiều cao Site trong ví dụ là legal placement row height cho ordinary single-height cells; chiều rộng Site là unit width nhỏ nhất mà một cell có thể chiếm trên placement grid.

Quan hệ giữa Site và Track: Track count mô tả routing-resource architecture bên trong/quanh standard-cell row; Site định nghĩa legal placement granularity. Mapping giữa track-count labels và Site height phụ thuộc library/PDK. [Needs verification]

Cell width constraint:
$$\text{Cell Width} = k \times \text{Site Width}, \quad k \in \mathbb{Z}^+$$

Cell height constraint:
$$\text{Cell Height} = m \times \text{Site Height}, \quad m \in \{1, 2, 4, \ldots\}$$

## Constrains
- **[[Row]]**: Row được tạo bằng cách tile Sites theo chiều ngang; Row height = Site height; Row là aggregation của Sites thành một horizontal strip
- **[[PlacementGrid]]**: Placement Grid = mạng lưới các Site positions trên toàn Core area; tất cả cells phải được placed tại Site-aligned positions
- **[[CellAbstract]]**: Macro LEF của mỗi Standard Cell reference Site name để declare "cell này thuộc loại Site nào"; Placement tool enforce Site alignment constraints dựa trên declaration này

## Requires
- [[LEF]] — Site definition nằm trong Tech LEF; không có Tech LEF, Placement tool không biết kích thước unit cell và alignment rules
- [[Pitch]] — Site width thường bằng M1 Pitch; Site height được thiết kế để contain đúng số M1 Tracks tương ứng với library track count (7-track, 9-track...); Pitch là foundation vật lý của Site dimensions

## Used by
- [[Row]] — Rows được formed bằng cách đặt Sites liền kề theo chiều ngang; "DO N STEP site_width" trong DEF Row definition lặp lại Site N lần
- [[PlacementGrid]] — Placement Grid = all Site positions trên Core area; Placement tool chỉ được đặt cell tại Site-aligned positions
- [[Placement]] — Placement Legalization step ensure tất cả cells được placed tại valid Site positions; cells placed off-Site trong Global Placement được snapped về nearest Site position
- [[CellAbstract]] — Macro LEF declares `SITE core` để Placement tool biết cell này cần align với "core" Site type; không có Site declaration, Placement tool cannot place cell correctly

## Key insight
Site là abstraction quan trọng giúp Library Design và Physical Design decoupled — Library designer chỉ cần đảm bảo cell dimensions tuân thủ Site constraints, PD engineer chỉ cần đảm bảo Placement align với Sites; hai bên không cần biết chi tiết implementation của nhau. Một số flow dùng multi-height/mixed-track libraries; rule phối hợp giữa các loại cell và Row topology phụ thuộc library policy/tool constraints của từng project. [Needs verification]

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Defined in: [[LEF]] — Tech LEF, SITE statement
→ Dimensions: Site width/height define legal placement granularity; relation to [[Track]] count depends on library/PDK [Needs verification]
→ Tiled into: [[Row]] (horizontal) → [[PlacementGrid]] (2D)
→ Referenced by: [[CellAbstract]] — SITE declaration
→ Enforced by: [[Placement]] — legalization step
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Row]] · [[PlacementGrid]]
