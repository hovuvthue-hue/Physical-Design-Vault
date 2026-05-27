---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement
used_by: [Placement, CTS, Routing, STA]
requires: [GlobalPlacement, Legalization, PlacementGrid, StandardCell]
chain: Chain_PnR_Flow
---
# DetailedPlacement

## Definition
**Detailed Placement** là giai đoạn refinement cục bộ sau [[GlobalPlacement]] và [[Legalization]], nhằm cải thiện chất lượng placement trong khi vẫn giữ legality.

## What it refines
Detailed Placement tập trung tinh chỉnh local:
- giảm wirelength cục bộ (thường nhìn qua proxy như [[HPWL]]),
- cải thiện phân bố [[PlacementDensity]] ở các vùng nhạy,
- cải thiện posture cho timing/congestion trước downstream steps.

## Relation to GlobalPlacement and Legalization
- [[GlobalPlacement]] tạo nghiệm vị trí gần đúng.
- [[Legalization]] đưa nghiệm đó về trạng thái hợp lệ.
- Detailed Placement tinh chỉnh thêm trên nghiệm đã legal để tăng QoR trước [[CTS]] và [[Routing]].

## Constraints preserved
Trong quá trình refine, các ràng buộc legal placement vẫn phải được giữ:
- align theo [[PlacementGrid]] / [[Site]] / [[Row]],
- không overlap,
- tôn trọng blockages và ranh giới placement.

Detailed Placement giúp cải thiện QoR, nhưng không được xem là bước có thể “chữa mọi vấn đề” timing/congestion.

## Requires
- [[GlobalPlacement]]
- [[Legalization]]
- [[PlacementGrid]]
- [[StandardCell]]

## Used by
- [[Placement]]
- [[CTS]]
- [[Routing]]
- [[STA]]

## Related
- [[PlacementBlockage]]
