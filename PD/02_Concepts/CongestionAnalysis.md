---
tags: [concept, pnr-flow, placement, routing]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [Placement, DetailedPlacement, GlobalRouting, Routing, Signoff]
requires: [PlacementDensity, RoutingGrid, PlacementBlockage, Placement, GlobalRouting]
chain: Chain_PnR_Flow
---
# CongestionAnalysis

## Definition
**CongestionAnalysis** là hoạt động ước lượng mức áp lực sử dụng tài nguyên routing sau [[Placement]], trước khi đi vào route chi tiết. Mục tiêu là dự báo vùng có nguy cơ **Routing Congestion** để kịp tối ưu lại placement/floorplan trước các bước downstream.

Đây là bước **analysis/estimation**, không phải quá trình tạo full route cuối cùng như [[Routing]]. Trong nội bộ Routing, [[GlobalRouting]] tiếp tục phơi bày congestion bằng cách so sánh routing demand với routing supply ở mức route-planning.

## What congestion means
Congestion xảy ra khi số routing tracks khả dụng tại một vùng layout nhỏ hơn số tracks cần thiết để route tất cả nets đi qua vùng đó.

**GCELL (Global Routing Cell)** là đơn vị phân tích congestion: tool chia Core area thành một lưới GCELLs thô, mỗi GCELL đại diện cho một vùng nhỏ chứa số Tracks nhất định theo hai chiều. Congestion được đo per GCELL bằng cách so sánh routing demand với routing supply trong từng GCELL.

**Overflow Score** là metric toàn cục đo congestion theo từng chiều:

$$\text{Overflow}_H = \text{Horizontal Tracks Available} - \text{Horizontal Tracks Required}$$

$$\text{Overflow}_V = \text{Vertical Tracks Available} - \text{Vertical Tracks Required}$$

Overflow Score được normalize thành phần trăm (range: 0% → 100%). Giá trị âm biểu thị thiếu hụt tracks. Hướng dẫn thực tế: **Overflow > 1% → design thường khó route**.

**Hotspot Score** đo mức độ nghiêm trọng của các vùng congestion cục bộ, tập trung vào các điểm xấu nhất thay vì trung bình toàn chip. Hướng dẫn thực tế: **Hotspot > 100 → vùng đó thường không thể route được**.

| Metric | Phạm vi | Ngưỡng thực tế |
|---|---|---|
| Overflow Score | Toàn chip, per chiều H/V | > 1% → typically difficult to route |
| Hotspot Score | Vùng cục bộ nghiêm trọng nhất | > 100 → typically not routable |

Hai metrics này bổ trợ nhau: Overflow phản ánh sức khỏe routing toàn chip, Hotspot chỉ ra vùng cần can thiệp cụ thể. Giá trị pass/fail signoff chính xác phụ thuộc tool, PDK và project methodology. [Needs verification]

## Why congestion analysis is needed after Placement
- Placement quyết định phân bố cell/pin, nên quyết định phần lớn routing pressure ban đầu.
- Phát hiện sớm hotspot giúp giảm rủi ro sang [[Routing]] mới phát hiện nghẽn nặng.
- Kết quả CongestionAnalysis là tín hiệu để quay lại vòng refine trong **Post-Placement Optimization** (dịch cell, tinh chỉnh blockage, hoặc floorplan refinement nếu cần).

## Relation to PlacementDensity and RoutingGrid
- [[PlacementDensity]] cao cục bộ thường kéo theo pin density cao, làm tăng routing demand.
- [[RoutingGrid]] mô tả hạ tầng supply (tracks/layer directions) mà demand phải "fit" vào; [[GlobalRouting]] dùng quan hệ demand/supply này để nhận diện vùng route plan có nguy cơ nghẽn.
- [[Track]] và [[Pitch]] ảnh hưởng trực tiếp routing capacity khả dụng theo vùng/layer.
- [[PlacementBlockage]] có thể giảm demand tại vùng nhạy, nhưng cũng có thể đẩy demand sang vùng lân cận nếu dùng không hợp lý.
- [[DetailedPlacement]] thường là nơi thực hiện refinement cục bộ trước khi đánh giá lại congestion.

## Common causes
- Mật độ cell cục bộ cao — quá nhiều Standard Cells trong vùng nhỏ.
- Pin density cao — nhiều kết nối tập trung trong vùng nhỏ, làm [[PinAccess]] khó hoặc buộc route phải detour.
- Standard Cells đặt gần Macros — vùng biên Macro tạo routing bottleneck.
- Pin density cao tại edge của Macros — nhiều kết nối tập trung vào cạnh Macro hẹp.
- Macro channels hẹp / narrow corridors — routing resources không đủ theo vùng.
- **Poor Floorplan** — Macro placement không hợp lý tạo vùng congestion mang tính cấu trúc (structural hotspots).
- **Poor synthesis** — Logic synthesis không physically-aware tạo ra topology phức tạp, fanout lớn, hoặc clustering logic không phù hợp với phân bố vật lý → routing demand tập trung.
- **Poor PG Grid strategy** — Quá nhiều PG stripes cục bộ chiếm routing tracks → giảm resources còn lại cho signal nets.
- Tương tác blockage chưa hợp lý giữa nhu cầu placement và nhu cầu route.

## Techniques to minimize congestion
Các biện pháp xử lý congestion tại post-placement stage (theo mức độ can thiệp tăng dần):

**Tại Placement level:**
- **Thêm Placement Blockages** (hard/partial/soft) tại vùng congested để giảm cell density cục bộ → [[PlacementBlockage]].
- **Thêm Halo quanh Macros/Memories** để duy trì keep-out region và đảm bảo routing channels → [[PlacementBlockage]].
- **Cell Padding** — thêm spacing constraint quanh các cell types có pin density cao (Flip-flop, MBFF, complex gate) để giảm local pin crowding → [[CellPadding]].
- **Chuyển sang congestion-driven placement** — khi design có đủ timing margin, chuyển từ timing-driven (mặc định; tối ưu wire length nhưng có thể tạo congestion) sang congestion-driven (spread cells ra, wire length dài hơn nhưng congestion giảm). Điều kiện áp dụng: design congested VÀ còn đủ positive timing slack.

**Tại Floorplan/PDN level:**
- **Điều chỉnh PG Grid** — giảm số PG stripes cục bộ tại vùng congested để giải phóng routing tracks cho signal nets; cần đảm bảo PG Grid vẫn đáp ứng EM và IR Drop requirements.
- **Update Floorplan** — di chuyển Macros để tạo routing corridors tốt hơn; đây là biện pháp tốn kém nhất nhưng cần thiết khi congestion có tính cấu trúc.

**Upstream:**
- **Physically-aware synthesis** — đưa floorplan/blockage data vào synthesis flow để netlist được tạo ra phù hợp hơn với physical layout intent, giảm topology gây congestion ngay từ đầu.

## Limits of congestion analysis
CongestionAnalysis chỉ là phép dự báo dựa trên placement và mô hình ước lượng tại thời điểm post-placement. Kết quả tốt **không đảm bảo tuyệt đối** rằng design sẽ route/timing close hoàn toàn ở các bước sau.

## Requires
- [[Placement]]
- [[PlacementDensity]]
- [[RoutingGrid]]
- [[PlacementBlockage]]
- [[GlobalRouting]]

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
- [[CellPadding]]
- [[PlacementBlockage]]
- [[Chain_PnR_Flow]]