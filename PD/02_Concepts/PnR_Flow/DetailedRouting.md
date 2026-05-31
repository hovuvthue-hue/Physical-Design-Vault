---
tags: [concept, pnr-flow, routing]
group: PnR Flow
defined_in: Routing
used_by: [ParasiticExtraction, Signoff, STA]
requires: [Routing, GlobalRouting, RoutingGrid, Track, Pitch, Via, LEF, DEF]
chain: Chain_PnR_Flow
---
# DetailedRouting

## Definition
DetailedRouting là stage trong [[Routing]] biến route plan trừu tượng từ [[GlobalRouting]] thành wire và [[Via]] geometry cụ thể trên các routing layers. Đây là nơi router chọn [[Track]] cụ thể, đặt đoạn metal, chuyển layer bằng Via, và tạo routed geometry có thể được kiểm tra bằng physical verification, extraction và timing analysis sau route.

Khác với GlobalRouting, DetailedRouting làm việc ở mức legal geometry: wire/via phải phù hợp với [[RoutingGrid]], [[Pitch]], [[PinAccess|pin-access]] constraints, obstruction/blockage và routing rules từ [[LEF]]/[[DEF]].

## Position inside Routing
DetailedRouting đứng sau [[GlobalRouting]] và trước [[ParasiticExtraction]]/[[Signoff]]. GlobalRouting đưa ra hướng đi ở mức planning; DetailedRouting cụ thể hóa hướng đi đó thành layout vật lý.

Khi DetailedRouting hoàn tất, database route chứa geometry thực tế hơn cho wires và vias. Geometry này là cơ sở để extract RC parasitics, chạy post-route STA và chuẩn bị các kiểm tra signoff.

## What DetailedRouting produces
DetailedRouting tạo hoặc hoàn thiện:

- Wire shapes trên các routing layers hợp lệ.
- [[Via]] tại các điểm chuyển layer hoặc truy cập pin phù hợp.
- Connectivity vật lý cho các nets theo netlist.
- Routed [[DEF]] hoặc route database chứa physical geometry dùng cho downstream tools.

Kết quả này không chỉ là kế hoạch; nó là layout routing cụ thể được dùng bởi [[ParasiticExtraction]], [[STA]] và [[Signoff]].

## Correctness concerns
Ở mức concept, DetailedRouting phải giải quyết hoặc tránh các vấn đề physical correctness như open, short, [[DRC|spacing/rule violation]] và [[PinAccess|pin-access failure]]. Tên check, rule deck, ngưỡng pass/fail và thứ tự repair phụ thuộc PDK/foundry/tool/signoff policy. [Needs verification]

[[AntennaEffect|Antenna-related handling]] có thể xuất hiện trong routing hoặc signoff flow. Card này chỉ ghi nhận rằng DetailedRouting tạo wire/via geometry có thể ảnh hưởng các kiểm tra downstream.

## Relation to ParasiticExtraction and Signoff
[[ParasiticExtraction]] cần routed wire/via geometry từ DetailedRouting để tính RC parasitics thực tế hơn so với estimate trước route. Sau đó [[STA]] dùng parasitics đã extract để đánh giá timing sau route.

[[Signoff]] dùng layout đã route làm nền cho các kiểm tra physical/timing/[[SignalIntegrity|SI]]/reliability/manufacturability. Vì vậy chất lượng DetailedRouting ảnh hưởng trực tiếp khả năng closure, nhưng bản thân DetailedRouting không thay thế signoff checks.

## Limits
- DetailedRouting không tự định nghĩa rule values; rule đến từ technology/library/signoff context.
- DetailedRouting không bảo đảm mọi signoff check đều sạch nếu chưa chạy các bước verification tương ứng.
- Các thuật toán rip-up/reroute, cost function, via optimization, pin-access repair và antenna repair là tool/flow-specific. [Needs verification]
- Các concept [[DRC]], [[PinAccess]], [[SignalIntegrity]] và [[AntennaEffect]] được tách riêng để card này không trở thành signoff/tool manual.

## Requires
- [[Routing]] — parent flow chứa DetailedRouting.
- [[GlobalRouting]] — cung cấp route plan trừu tượng để detailed route cụ thể hóa.
- [[RoutingGrid]] — không gian hợp lệ để đặt wire/via geometry.
- [[Track]] — vị trí center-line hợp lệ cho wire trên từng layer.
- [[Pitch]] — khoảng cách track/layer ảnh hưởng legal spacing và routing capacity.
- [[Via]] — phần tử kết nối dọc giữa các metal layers.
- [[LEF]] — mô tả layer, pin, obstruction và routing/via rules ở mức abstract.
- [[DEF]] — chứa design placement/routing database và có thể lưu routed output.

## Used by
- [[ParasiticExtraction]] — đọc routed wire/via geometry để extract parasitics.
- [[Signoff]] — kiểm tra layout/timing/reliability dựa trên routed design.
- [[STA]] — dùng parasitics sau route để phân tích timing thực tế hơn.

## Related
→ Upstream stage: [[GlobalRouting]]
→ Parent flow: [[Routing]]
→ Geometry model: [[RoutingGrid]] · [[Track]] · [[Pitch]] · [[Via]]
→ File context: [[LEF]] · [[DEF]]
→ Downstream: [[ParasiticExtraction]] · [[STA]] · [[Signoff]]
→ Constraints context: [[RoutingBlockage]] · [[CongestionAnalysis]] · [[DRC]] · [[PinAccess]] · [[SignalIntegrity]] · [[AntennaEffect]]
