---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement
used_by: [Placement, GlobalPlacement, STA]
requires: [Placement, StandardCell]
chain: Chain_PnR_Flow
---
# HPWL

## Definition
**HPWL (Half-Perimeter Wirelength)** là metric ước tính wirelength nhanh ở giai đoạn Placement, dựa trên bounding box của các pins thuộc cùng một net.

## Why HPWL is used
- Tính nhanh, chi phí thấp, phù hợp để tool tối ưu lặp nhiều vòng trong [[Placement]] và [[GlobalPlacement]].
- Cho tín hiệu định hướng sớm về xu hướng wirelength trước khi có route chi tiết.
- Hữu ích cho pre-route QoR so sánh giữa các phương án placement.

## Formula
Với một net có tập pin tại các tọa độ (x, y):

`HPWL(net) = (x_max - x_min) + (y_max - y_min)`

Tổng HPWL của design thường là tổng HPWL trên các net được xét.

## What HPWL misses
HPWL không mô hình hóa đầy đủ route thực tế. Nó bỏ qua hoặc chỉ phản ánh gián tiếp:
- chướng ngại vật layout và blockages,
- routing detour,
- layer assignment,
- local congestion,
- topology route thực (không phải chỉ bounding box).

Vì vậy HPWL là proxy nhanh, không thay thế wirelength sau [[Routing]].

## Requires
- [[Placement]]
- [[StandardCell]]

## Used by
- [[Placement]]
- [[GlobalPlacement]]
- [[STA]] (pre-route estimate context)

## Related
- [[DetailedPlacement]]
- [[Routing]]
