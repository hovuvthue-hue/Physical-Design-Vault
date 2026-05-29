---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement
used_by: [Placement, Legalization, DetailedPlacement]
requires: [Placement, StandardCell, PlacementGrid, PlacementDensity, HPWL]
chain: Chain_PnR_Flow
---
# GlobalPlacement

## Definition
**Global Placement** là pha placement thô/coarse, tối ưu vị trí tương đối của cell trước khi cưỡng bức toàn bộ ràng buộc legality cuối cùng.

## Objectives
Các mục tiêu thường được cân bằng đồng thời:
- giảm wirelength proxy (thường qua [[HPWL]]),
- cải thiện timing posture,
- trải đều [[PlacementDensity]],
- giảm nguy cơ congestion cho bước [[Routing]] sau này.

## Why overlap may appear temporarily
Trong lúc tối ưu, tool có thể cho phép overlap tạm thời ở mức mô hình để tìm nghiệm tốt hơn theo multi-objective.

Điều này không có nghĩa output đã legal; overlap sẽ được xử lý ở bước sau.

## Output to Legalization / DetailedPlacement
Output của [[GlobalPlacement]] là tọa độ gần đúng (approximate positions) cho các cell.

Các tọa độ này cần qua [[Legalization]] và [[DetailedPlacement]] để:
- hết overlap,
- align đúng [[PlacementGrid]] (site/row legality),
- tinh chỉnh local QoR.

## Requires
- [[Placement]]
- [[StandardCell]]
- [[PlacementGrid]]
- [[PlacementDensity]]
- [[HPWL]]

## Used by
- [[Legalization]]
- [[DetailedPlacement]]
- [[Placement]]

## Related
- [[Routing]]
