---
tags: [concept, sta-timing, routing, parasitics]
group: STA — Timing
defined_in: ParasiticExtraction
used_by: [ParasiticExtraction, SPEF, STA, NetDelay, PostRouteOptimization, Signoff]
requires: [Routing, MetalStack, ITF, SPEF]
chain: Chain_STA_Basics
---
# InterconnectRC

## Definition
InterconnectRC là resistance và capacitance do routed interconnect tạo ra, bao gồm wire segments và [[Via]] trên các layers của [[MetalStack]]. Đây là cầu nối giữa geometry vật lý sau [[Routing]] và timing analysis trong [[STA]].

Trước khi có Routing hoàn chỉnh, RC của net thường chỉ là ước lượng dựa trên placement, topology dự đoán, hoặc routing model. Sau Routing, wire/via geometry thực tế cho phép [[ParasiticExtraction]] tính parasitics cụ thể hơn và ghi ra [[SPEF]].

## Why InterconnectRC matters
[[NetDelay]] không chỉ đến từ cell driver mà còn đến từ mạng RC của interconnect. Nếu routed wire dài hơn dự kiến, phải đi qua nhiều layer change, hoặc chạy gần net khác trong vùng congestion, InterconnectRC có thể làm delay, [[Slew]], và [[Slack]] thay đổi so với timing trước route.

Vì vậy post-route STA cần extracted RC thay vì chỉ dựa vào estimate. InterconnectRC là phần vật lý khiến cùng một logic net có timing khác nhau tùy theo vị trí placement, layer assignment, detour, via usage, và neighboring wires.

## Sources of R and C
Các thành phần chính ở mức khái niệm gồm:

- **Wire resistance** — phụ thuộc vào material và geometry của wire trên từng metal layer.
- **Wire capacitance** — gồm tương tác với môi trường xung quanh wire, substrate, và các layer lân cận.
- **Via resistance/capacitance** — phát sinh tại các điểm net đổi layer qua [[Via]].
- **Coupling capacitance** — tương tác điện dung giữa các nets gần nhau hoặc chạy song song; card này giữ coupling ở mức khái niệm và không tách thành card riêng.

Các model chi tiết, hệ số extraction, và cách phân tách capacitance theo tool/PDK/process corner là flow-specific [Needs verification].

## Estimated RC vs actual extracted RC
Estimated RC trước Routing dùng thông tin chưa hoàn chỉnh: khoảng cách giữa pins, congestion estimate, hoặc routing prediction. Nó hữu ích để guide placement/CTS/optimization nhưng chưa biết chính xác layer, track, detour, và via count cuối cùng.

Actual extracted RC sau Routing dùng routed wire/via geometry thật. [[ParasiticExtraction]] kết hợp geometry này với [[ITF]] và technology data để tạo [[SPEF]], giúp STA phân tích timing gần với layout đã route hơn.

## Relation to SPEF and STA
[[SPEF]] là representation trao đổi của extracted parasitics. STA đọc SPEF để annotate RC vào timing graph, từ đó tính [[NetDelay]], cập nhật [[Slew]], và đánh giá [[Slack]] cho setup/hold/signoff. [[PostRouteOptimization]] dùng bối cảnh này để cleanup timing/DRV dựa trên parasitics hậu route thay vì estimate trước route.

Nếu InterconnectRC không được extract đúng corner hoặc không khớp với routed layout, kết quả STA có thể không phản ánh silicon/layout thực tế [Needs verification]. Vì vậy InterconnectRC nằm giữa Routing, ParasiticExtraction, SPEF, và Signoff.

## Trade-offs / limits
Giảm resistance bằng cách chọn layer phù hợp, rút ngắn path, hoặc giảm layer changes có thể cải thiện timing nhưng bị giới hạn bởi routing resource và congestion. Tăng spacing có thể giảm coupling risk nhưng tiêu thụ thêm routing capacity. Thêm redundant/multi-cut Via có thể giúp reliability trong một số flow, nhưng cũng dùng thêm diện tích routing và phụ thuộc policy cụ thể [Needs verification].

InterconnectRC vì vậy không phải một con số độc lập; nó là kết quả của trade-off giữa geometry, routing resources, electrical behavior, và manufacturability.

## Requires
- [[Routing]] — tạo wire/via geometry thực tế.
- [[MetalStack]] — xác định layer hierarchy và môi trường vật lý của interconnect.
- [[ITF]] — cung cấp tham số công nghệ cho extraction.
- [[SPEF]] — lưu và trao đổi parasitics đã extract cho timing analysis.

## Used by
- [[ParasiticExtraction]] — tạo InterconnectRC từ routed geometry và technology data.
- [[SPEF]] — biểu diễn extracted resistance/capacitance theo net.
- [[STA]] — dùng RC đã annotate để tính timing.
- [[NetDelay]] — phụ thuộc trực tiếp vào RC của routed interconnect.
- [[PostRouteOptimization]] — dùng extracted RC để guide cleanup hậu route có kiểm soát.
- [[Signoff]] — dùng extracted RC để kiểm tra timing và các phân tích hậu route liên quan.

## Related
→ Physical source: [[Routing]] · [[MetalStack]] · [[Via]]
→ Extraction path: [[ParasiticExtraction]] → [[SPEF]]
→ Timing impact: [[STA]] · [[NetDelay]] · [[Slew]] · [[Slack]] · [[PostRouteOptimization]]
→ Technology data: [[ITF]]
