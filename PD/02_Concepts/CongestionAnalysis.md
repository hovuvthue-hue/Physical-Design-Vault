---
tags: [concept, pnr-flow, placement, routing]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [Placement, DetailedPlacement, GlobalRouting, Routing, Signoff]
requires: [PlacementDensity, RoutingGrid, PlacementBlockage, Placement]
chain: Chain_PnR_Flow
---
# CongestionAnalysis

## Definition
**CongestionAnalysis** là hoạt động ước lượng mức áp lực sử dụng tài nguyên routing sau [[Placement]], trước khi đi vào route chi tiết. Mục tiêu là dự báo vùng có nguy cơ **Routing Congestion** để kịp tối ưu lại placement/floorplan trước các bước downstream.

Đây là bước **analysis/estimation**, không phải quá trình tạo full route cuối cùng như [[Routing]]. Trong nội bộ Routing, [[GlobalRouting]] tiếp tục phơi bày congestion bằng cách so sánh routing demand với routing supply ở mức route-planning.

## What congestion means
Ở mức concept, congestion xảy ra khi routing demand cục bộ có xu hướng vượt quá routing supply khả dụng trong một vùng layout.

Các chỉ số cụ thể (ví dụ tên metric, cách normalize, ngưỡng pass/fail signoff) phụ thuộc tool/PDK/flow và cần được xác nhận theo dự án thực tế. [Needs verification]

Numeric examples như overflow khoảng 1% hoặc hotspot score khoảng 100 nên được hiểu là ví dụ tool/report hoặc mnemonic reference, không phải universal pass/fail rules. Cách diễn giải phụ thuộc tool định nghĩa overflow/hotspot như thế nào, GCELL size, routing-resource model, PDK, floorplan, và project flow. [Needs verification]

## Why congestion analysis is needed after Placement
- Placement quyết định phân bố cell/pin, nên quyết định phần lớn routing pressure ban đầu.
- Phát hiện sớm hotspot giúp giảm rủi ro sang [[Routing]] mới phát hiện nghẽn nặng.
- Kết quả CongestionAnalysis là tín hiệu để quay lại vòng refine trong **Post-Placement Optimization** (dịch cell, tinh chỉnh blockage, hoặc floorplan refinement nếu cần).

## Relation to PlacementDensity and RoutingGrid
- [[PlacementDensity]] cao cục bộ thường kéo theo pin density cao, làm tăng routing demand.
- [[RoutingGrid]] mô tả hạ tầng supply (tracks/layer directions) mà demand phải “fit” vào; [[GlobalRouting]] dùng quan hệ demand/supply này để nhận diện vùng route plan có nguy cơ nghẽn.
- [[Track]] và [[Pitch]] ảnh hưởng trực tiếp routing capacity khả dụng theo vùng/layer.
- [[PlacementBlockage]] có thể giảm demand tại vùng nhạy, nhưng cũng có thể đẩy demand sang vùng lân cận nếu dùng không hợp lý.
- [[DetailedPlacement]] thường là nơi thực hiện refinement cục bộ trước khi đánh giá lại congestion.

## Common causes
- Mật độ cell cục bộ cao.
- Pin density cao (nhiều kết nối tập trung trong vùng nhỏ), có thể làm [[PinAccess]] khó hơn hoặc buộc route phải detour.
- Macro channels hẹp / narrow corridors.
- Routing resources không đủ (theo grid/layer khả dụng).
- Tương tác blockage chưa hợp lý giữa nhu cầu placement và nhu cầu route.

## Typical conceptual responses
- Spread cells để giảm local density.
- Điều chỉnh [[PlacementBlockage]] (thêm/bớt/đổi chiến lược vùng hạn chế).
- Rà soát lại macro channels nếu nghẽn lặp lại theo cấu trúc floorplan.
- Re-run placement refinement và đánh giá congestion lại trước khi handoff sang [[Routing]].

## Limits of congestion analysis
CongestionAnalysis chỉ là phép dự báo dựa trên placement và mô hình ước lượng tại thời điểm post-placement. Kết quả tốt **không đảm bảo tuyệt đối** rằng design sẽ route/timing close hoàn toàn ở các bước sau.

## Requires
- [[Placement]]
- [[PlacementDensity]]
- [[RoutingGrid]]
- [[PlacementBlockage]]

## Used by
- [[Placement]]
- [[DetailedPlacement]]
- [[GlobalRouting]]
- [[PinAccess]]
- [[Routing]]
- [[Signoff]]

## Related
- [[Track]]
- [[Pitch]]
- [[GlobalRouting]]
- [[PinAccess]]
- [[Routing]]
- [[DetailedPlacement]]
- [[Chain_PnR_Flow]]
