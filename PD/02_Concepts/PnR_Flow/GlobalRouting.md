---
tags: [concept, pnr-flow, routing]
group: PnR Flow
defined_in: Routing
used_by: [DetailedRouting, CongestionAnalysis, Routing, Signoff]
requires: [Routing, RoutingGrid, Track, Pitch, LEF, DEF]
chain: Chain_PnR_Flow
---
# GlobalRouting

## Early Global Routing during Placement

GlobalRouting cũng được gọi explicitly trong **Placement flow** (Innovus: `route_early_global`) để estimate routing feasibility trong khi đặt cells — trước khi GlobalRouting stage chính thức sau CTS. Đây là predictive step giúp placer đưa ra quyết định tốt hơn về cell distribution và congestion.

Trong context này:
- Results được đánh dấu status **"unknown"** — đây là estimate, không phải final route geometry
- **Overflow (%)** là key metric: tỷ lệ routing demand vượt quá routing supply cục bộ
  - Overflow < 1% → generally routable
  - Overflow > 1% → likely routing challenges → cần điều chỉnh placement trước khi tiến sang CTS/Routing
- Cells không được routed thực tế — đây là coarse predictive step

**Phân biệt với GlobalRouting chính thức:** Early GR trong Placement = estimation/feedback tool cho placer; GR sau CTS = route planning stage trong Routing flow (output là route plan hướng dẫn DetailedRouting).

## Definition
GlobalRouting là stage lập kế hoạch routing ở mức trừu tượng trước khi hình học wire/via hợp lệ cuối cùng được tạo ra. Thay vì quyết định từng đoạn metal chính xác trên từng [[Track]], GlobalRouting ước lượng topology, hướng đi tương đối, nhu cầu dùng tài nguyên routing và rủi ro congestion để chuẩn bị cho [[DetailedRouting]].

Ở mức concept, GlobalRouting có thể chia layout thành các vùng routing thô hoặc resource buckets để so sánh routing demand với routing supply. Cách chia vùng, tên bucket và thuật toán cụ thể phụ thuộc tool/flow, nên cần được xem như implementation detail nếu đi sâu hơn. [Needs verification]

## Position inside Routing
Trong [[Routing]], GlobalRouting đứng sau routing setup và trước [[DetailedRouting]]. Nó nhận database sau placement/CTS cùng [[LEF]], [[DEF]], [[RoutingGrid]], [[Track]], [[Pitch]] và các constraint liên quan để tạo route plan trừu tượng.

Output của GlobalRouting hướng dẫn DetailedRouting chọn vùng/layer/đường đi hợp lý hơn, nhưng output này chưa phải layout đã route xong, chưa phải final legal wire/via geometry, và không tự bảo đảm DRC-clean hay signoff-clean.

## What GlobalRouting estimates
GlobalRouting thường ước lượng ở mức routing-resource planning:

- Topology hoặc hướng kết nối gần đúng cho các nets.
- Routing demand theo vùng/layer ở mức coarse.
- Routing supply khả dụng dựa trên [[RoutingGrid]], [[Track]], [[Pitch]] và các vùng bị hạn chế.
- Mức sử dụng routing resource để phát hiện hotspot có nguy cơ nghẽn.
- Trade-off giữa đường đi ngắn cho timing và đường đi tránh nghẽn cho routability.

Các quyết định timing/routability ở đây vẫn là quyết định concept-level; độ chính xác cuối cùng phụ thuộc routed geometry sau DetailedRouting và extraction.

## Relation to congestion
[[CongestionAnalysis]] và GlobalRouting cùng nhìn vào quan hệ demand/supply của routing resource. CongestionAnalysis giúp phát hiện rủi ro từ placement/floorplan, còn GlobalRouting làm rõ hơn nơi route demand có thể vượt supply khi lập route plan.

[[RoutingBlockage]] và [[PlacementBlockage]] có thể làm thay đổi supply hoặc phân bố demand: RoutingBlockage trực tiếp hạn chế tài nguyên route trên vùng/layer, còn PlacementBlockage có thể làm thay đổi mật độ cell/pin và luồng kết nối đi qua vùng lân cận.

## Relation to DetailedRouting
[[DetailedRouting]] sử dụng kế hoạch từ GlobalRouting để tạo wire/via geometry cụ thể trên [[RoutingGrid]]. Nếu GlobalRouting chọn hướng đi quá nghẽn hoặc đánh giá supply quá lạc quan, DetailedRouting có thể phải detour, rip-up/reroute, hoặc để lại vấn đề correctness cần xử lý tiếp.

Ngược lại, một route plan tốt giúp DetailedRouting có không gian hợp lệ hơn để hoàn tất connectivity, giảm detour không cần thiết và chuẩn bị dữ liệu tốt hơn cho [[ParasiticExtraction]] và [[Signoff]].

## Limits
- GlobalRouting không tạo final routed [[DEF]] với đầy đủ wire/via geometry hợp lệ.
- GlobalRouting không thay thế DetailedRouting, DRC checking, extraction hoặc Signoff.
- Kết quả congestion tốt ở GlobalRouting không bảo đảm tuyệt đối rằng design sẽ route-clean ở detailed level.
- Các chi tiết như thuật toán search, grid bucket, cost function, layer preference hoặc repair policy là tool/flow-specific. [Needs verification]

## Requires
- [[Routing]] — context flow chứa GlobalRouting và DetailedRouting.
- [[RoutingGrid]] — mô hình tài nguyên routing mà route plan phải dùng.
- [[Track]] — đơn vị tài nguyên line-level mà DetailedRouting sẽ cụ thể hóa sau đó.
- [[Pitch]] — ảnh hưởng mật độ Track và supply routing theo layer/vùng.
- [[LEF]] — cung cấp layer/routing rule/pin abstract cần cho planning.
- [[DEF]] — cung cấp design database, placement và các thông tin layout đã instantiate.

## Used by
- [[DetailedRouting]] — dùng route plan để tạo wire/via geometry hợp lệ.
- [[CongestionAnalysis]] — đối chiếu hotspot và demand/supply ở mức route planning.
- [[Routing]] — dùng GlobalRouting như stage nội bộ giữa setup và detailed route.
- [[Signoff]] — gián tiếp phụ thuộc chất lượng route plan vì route plan ảnh hưởng routed geometry downstream.

## Related
→ Parent flow: [[Routing]]
→ Downstream stage: [[DetailedRouting]]
→ Resource model: [[RoutingGrid]] · [[Track]] · [[Pitch]]
→ Congestion controls: [[CongestionAnalysis]] · [[RoutingBlockage]] · [[PlacementBlockage]]
→ Handoff context: [[ParasiticExtraction]] · [[Signoff]]
