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

## Two variants of Placement Density

**Cell Density** — chỉ tính Standard Cells:
$$\text{Cell Density} = \frac{\text{Total Standard Cell Area}}{\text{Total Placeable Area}}$$

**Core Density** — tính cả Macros:
$$\text{Core Density} = \frac{\text{Total Standard Cell Area} + \text{Macro Area}}{\text{Total Placeable Area}}$$

Cell Density loại trừ Macros nên phản ánh mức độ đặc của Standard Cell fabric; Core Density cho bức tranh tổng thể của Core area.

## Practical guideline
Placement density thường được giữ dưới **~70%** để đảm bảo routability. Density cao hơn dẫn đến tăng congestion và khó routing.
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

Vì vậy density control và blockage strategy luôn đi cùng nhau trong [[GlobalPlacement]], và thường được kiểm tra qua [[CongestionAnalysis]].

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
- [[CongestionAnalysis]]
