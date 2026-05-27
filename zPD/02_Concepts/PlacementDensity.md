---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement
used_by: [Placement, GlobalPlacement, Routing]
requires: [Placement, PlacementGrid, PlacementBlockage, StandardCell]
chain: Chain_PnR_Flow
---
# PlacementDensity

## Definition
**Placement Density** mô tả mức độ diện tích [[StandardCell]] chiếm dụng trên một vùng diện tích placement hợp lệ.

Nói ngắn gọn: vùng càng nhiều cell trên cùng legal area thì density càng cao.

## Local density vs global utilization
- **Global utilization**: góc nhìn tổng thể toàn core/placeable area.
- **Local density**: góc nhìn theo từng vùng nhỏ trong layout.

Một design có global utilization “ổn” vẫn có thể gặp hotspot nếu local density cao không đều.

## Why density matters
- Local density cao thường kéo theo pin density cao.
- Pin density cao làm routing demand tăng, dễ tăng congestion risk.
- Density phân bố tốt giúp cân bằng giữa wirelength, timing posture và khả năng routability.

## Relation to blockages and congestion
[[PlacementBlockage]] thay đổi legal placement availability theo vùng, nên ảnh hưởng trực tiếp profile density:
- Hard/partial/soft blockage có thể ép cell dồn sang vùng khác.
- Nếu dồn không hợp lý, local density tăng và congestion tăng ở khu vực lân cận.

Vì vậy density control và blockage strategy luôn đi cùng nhau trong [[GlobalPlacement]].

## Requires
- [[Placement]]
- [[PlacementGrid]]
- [[PlacementBlockage]]
- [[StandardCell]]

## Used by
- [[Placement]]
- [[GlobalPlacement]]
- [[Routing]]

## Related
- [[HPWL]]
- [[DetailedPlacement]]
