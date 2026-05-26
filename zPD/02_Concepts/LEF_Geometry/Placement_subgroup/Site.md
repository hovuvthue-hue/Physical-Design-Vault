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
Site là đơn vị diện tích nhỏ nhất (minimum placement unit) được Foundry định nghĩa trong Tech LEF, đóng vai trò là "ô Lego" cơ sở trên bề mặt chip cho Placement. Mọi Standard Cell bắt buộc phải có chiều cao bằng đúng chiều cao của Site (hoặc bội số nguyên lần cho double-height cells) và chiều rộng là bội số nguyên của chiều rộng Site. Site đảm bảo tính regularity của Placement — tất cả cells align trên cùng một grid, cho phép xếp chúng liền kề thành các Rows liên tục mà không có khoảng trống hay overlap.

## Computed from
Site được Foundry định nghĩa trong Tech LEF:

SITE core 
SIZE 0.140 BY 0.900 ; 
SYMMETRY Y ; 
CLASS CORE ; 
END core

Chiều cao Site (0.900um trong ví dụ) được Foundry thiết kế để accommodate: VDD power rail + VSS power rail + N-Well region + P-Substrate region + đủ số M1 Tracks cho routing bên trong cell. Chiều rộng Site (0.140um) = 1 M1 Pitch — đây là unit width nhỏ nhất mà một cell có thể có.

Quan hệ giữa Site và Track:
$$\text{Site Height} = N_{tracks} \times \text{M1 Pitch} + \text{Power Rail Width} \times 2$$

Ví dụ: TSMC 22nm "9-track" library → Site height = 9 × M1 Pitch + power rails overhead.

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
[USER REVIEW — draft suggestion]: Site là abstraction quan trọng giúp Library Design và Physical Design decoupled — Library designer chỉ cần đảm bảo cell dimensions tuân thủ Site constraints, PD engineer chỉ cần đảm bảo Placement align với Sites; hai bên không cần biết chi tiết implementation của nhau. Điểm thực tế: "7-track vs 9-track cell library" thực chất là "Site height = 7 M1 Pitches vs Site height = 9 M1 Pitches" — đây là trade-off giữa area (7-track nhỏ hơn, tốt cho dense logic) và performance (9-track có transistor rộng hơn, drive strength mạnh hơn). Khi một chip cần cả hai, designer dùng multi-height cell strategy: 7.5-track cells cho non-critical logic, 9-track cells cho critical paths — nhưng cần careful floorplanning vì double-height cells không thể placed trong rows designed cho single-height.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Defined in: [[LEF]] — Tech LEF, SITE statement
→ Dimensions: Width = 1 M1 [[Pitch]] · Height = N × M1 [[Pitch]]
→ Tiled into: [[Row]] (horizontal) → [[PlacementGrid]] (2D)
→ Referenced by: [[CellAbstract]] — SITE declaration
→ Enforced by: [[Placement]] — legalization step
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Row]] · [[PlacementGrid]]