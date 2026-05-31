---
tags: [concept, lef-geometry]
group: LEF Geometry — Placement
defined_in: Derived từ Row tiling trong Core area — không có section riêng trong DEF, được implicit bởi ROWS
used_by: [Placement, Floorplanning, CellAbstract]
requires: [Row, Site, DEF]
chain: Chain_LEF_to_PnR
---
# PlacementGrid

## Definition
Placement Grid là toàn bộ mạng lưới các valid placement positions trong Standard Cell placement area, được formed bằng cách tile tất cả [[Row]] theo chiều dọc. Mỗi [[Site]] position trong mỗi Row là một node trên Placement Grid — đây là vị trí hợp lệ để một Standard Cell được placed. Placement Grid đối với [[Placement]] giống như Routing Grid đối với [[Routing]]: nó discretize không gian liên tục thành tập hợp finite valid positions.
Trong thực thi Placement, mọi cell cuối cùng phải snap về các vị trí hợp lệ này (site/row aligned) ở bước legalization; PlacementGrid là điều kiện legality, không phải bảo đảm trực tiếp cho routability.

## Computed from
Placement Grid không có explicit representation riêng — nó được implied bởi tập hợp tất cả Rows trong DEF:

$$\text{Placement Grid} = \bigcup_{\text{all Rows}} \{\text{Site positions in Row}\}$$

Số lượng valid placement positions:
$$N_{positions} = N_{rows} \times N_{sites/row} = \frac{\text{Core Height}}{\text{Site Height}} \times \frac{\text{Core Width}}{\text{Site Width}}$$

Không phải tất cả positions đều available — các positions có thể bị loại khỏi legal placement do Macro footprints (không có Rows), vùng cấm/giới hạn placement, hoặc các ràng buộc physical khác của floorplan.

**Quan hệ cốt lõi — KEY CONSTRAINT trong Physical Design:**
$$\text{Routing Grid} \leftrightarrow \text{Placement Grid must align}$$

PlacementGrid và RoutingGrid phải tương thích để StandardCell pins có thể được router access hợp lệ sau placement. Site định nghĩa legal placement granularity, còn Track/Pitch định nghĩa routing-resource geometry. M1/M2 pin access phụ thuộc vào cell abstract pin shapes, Track/Pitch, routing layers, Site/Row alignment và library/PDK rules; exact Site-to-track mapping là library/PDK-dependent. [Needs verification]

## Constrains
- Legal placement theo PlacementGrid là prerequisite cho downstream [[Placement]] và [[Routing]], nhưng tự nó không đảm bảo design sẽ routable hoặc timing-clean.
- **[[Placement]]**: Placement tool search space = Placement Grid; cells phải placed tại valid grid positions; Global Placement có thể violate grid tạm thời nhưng Legalization bắt buộc snap tất cả cells về nearest valid Site position
- **[[Floorplanning]]**: Placement Grid density (số valid positions) xác định Core area utilization capacity; Floorplanning phải ensure Core area đủ lớn để accommodate tất cả Standard Cells với target utilization ratio
- **[[CellAbstract]]**: Cell width phải là bội số của Site width để cell fit exactly vào N consecutive positions trên Placement Grid; off-grid cell dimensions = library error

## Requires
- [[Row]] — Placement Grid = tập hợp tất cả Site positions trong tất cả Rows; không có Rows, không có Placement Grid
- [[Site]] — Site là unit của Placement Grid; Site dimensions quyết định grid resolution (granularity của placement decisions)
- [[DEF]] — Rows (và do đó Placement Grid) được stored và defined trong DEF ROWS section post-Floorplan; Placement tool reads DEF để build internal Placement Grid data structure

## Used by
- [[Placement]] — Placement tool internal model = Placement Grid; tất cả placement decisions (initial placement, optimization moves, legalization) operate trên grid positions; timeliness của Placement directly proportional to grid resolution
- [[Floorplanning]] — Floorplanning generates Placement Grid bằng cách instantiate Rows; Floorplan quality metrics như Row utilization và Macro placement feasibility đều reference Placement Grid
- [[CellAbstract]] — Standard Cell Pin positions phải align với Placement Grid → Routing Grid intersection để ensure Router có thể access Pin bằng Via drop mà không cần off-grid jogs; đây là pin accessibility requirement

## Key insight
PlacementGrid và RoutingGrid phải tương thích để StandardCell pins có thể được router access hợp lệ sau placement. Site/Row xác định legal placement granularity, còn Track/Pitch xác định routing-resource geometry. Sự tương thích giữa Site dimensions, pin shapes, routing tracks và layer rules là property của library/PDK ecosystem; không nên coi Site height hoặc Site width là universal formula theo M1 Pitch. [Needs verification]

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formed from: all [[Row]]s tiling Core area vertically
→ Unit: [[Site]] position
→ KEY CONSTRAINT: must align with [[RoutingGrid]] — same origin, compatible spacing
→ Compatibility mechanism: Site/PlacementGrid phải tương thích với [[RoutingGrid]] theo library/PDK rules [Needs verification]
→ Stored implicitly in: [[DEF]] ROWS section
→ Consumed by: [[Placement]] · [[Floorplanning]]
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Site]] · [[Row]]
