---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement
used_by: [Placement, DetailedPlacement, CTS]
requires: [GlobalPlacement, PlacementGrid, Site, Row, PlacementBlockage]
chain: Chain_PnR_Flow
---
# Legalization

## Definition
**Legalization** là bước chuyển kết quả từ [[GlobalPlacement]] thành placement hợp lệ về mặt hình học/vật lý.

## What legality means
Một placement được coi là legal khi thỏa các điều kiện cốt lõi:
- cell align đúng [[Site]],
- cell nằm hợp lệ trong [[Row]],
- không overlap nhau,
- không xâm phạm vùng bị ràng buộc bởi macro hoặc [[PlacementBlockage]].

## Why legalization is needed after GlobalPlacement
[[GlobalPlacement]] tập trung tối ưu thô nên có thể để lại overlap hoặc lệch grid.

Legalization là cầu nối bắt buộc để đưa design về trạng thái có thể dùng cho các bước refinement và downstream analysis.

## Movement vs legality tradeoff
Mục tiêu thực tế là **hợp pháp hóa với mức dịch chuyển nhỏ nhất có thể** từ nghiệm global placement.

Dịch chuyển quá mạnh có thể làm xấu wirelength/timing posture; dịch chuyển quá ít có thể không giải quyết triệt để legality.

## Requires
- [[GlobalPlacement]]
- [[PlacementGrid]]
- [[Site]]
- [[Row]]
- [[PlacementBlockage]]

## Used by
- [[DetailedPlacement]]
- [[Placement]]
- [[CTS]]

## Related
- [[HPWL]]
- [[PlacementDensity]]
