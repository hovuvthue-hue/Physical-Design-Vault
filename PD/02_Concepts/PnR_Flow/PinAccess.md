---
tags: [concept, pnr-flow, routing]
group: PnR Flow
defined_in: DetailedRouting
used_by: [DetailedRouting, Routing, DRC, Signoff]
requires: [Pin, LEF, RoutingGrid, Track, Pitch]
chain: Chain_PnR_Flow
---
# PinAccess

## Definition
PinAccess là khả năng router tiếp cận và kết nối hợp lệ tới [[Pin]] của standard cell hoặc macro. Một pin có PinAccess tốt khi router có thể đặt wire/via đến pin đó mà vẫn tôn trọng [[LEF]], [[RoutingGrid]], [[Track]], [[Pitch]], blockage/obstruction và rule context liên quan.

PinAccess là concept về routing feasibility tại vùng rất cục bộ quanh pin; nó không chỉ phụ thuộc vị trí pin mà còn phụ thuộc placement xung quanh, track availability và via access.

## Why PinAccess matters
Nếu router không thể access pin hợp lệ, [[DetailedRouting]] có thể phải detour, rip-up/reroute, dùng layer khác, hoặc để lại lỗi connectivity/DRC cần sửa ở downstream stage. Vì pin là điểm vào/ra của cell, lỗi PinAccess có thể biến một vùng placement tưởng như hợp lệ thành vùng khó route khi đi vào detail route.

Các metric pin-access, access point scoring hoặc rule về via-on-pin phụ thuộc library/PDK/tool/flow. [Needs verification]

## Main causes of poor pin access
- Placement quá dày làm nhiều pins cạnh tranh cùng routing resources.
- Pin shape nhỏ, phức tạp hoặc khó align với routing tracks.
- Obstruction, [[RoutingBlockage]] hoặc macro boundary che mất đường tiếp cận.
- Ít local [[Track]] khả dụng do [[Pitch]], layer direction hoặc routing demand xung quanh.
- Interface giữa macro và standard-cell rows tạo narrow channel hoặc pin crowding.
- Via access bị giới hạn bởi layer stack, enclosure hoặc cut/via rules. [Needs verification]

## Relation to DetailedRouting
[[DetailedRouting]] là nơi PinAccess trở thành vấn đề cụ thể: router phải chọn track, đặt wire end hoặc [[Via]], và kết nối đến pin shape mà không tạo [[DRC]] violation. Global route có thể cho thấy vùng route khả thi, nhưng detail route mới kiểm tra được việc access từng pin ở mức geometry.

PinAccess liên quan trực tiếp tới [[Pin]], [[LEF]], [[RoutingGrid]], [[Track]], [[Pitch]], [[RoutingBlockage]], [[PlacementDensity]] và [[CongestionAnalysis]].

## Impact on routing quality
PinAccess kém có thể gây:

- detour trong DetailedRouting;
- tăng rủi ro [[DRC]] do spacing/via/enclosure khó thỏa;
- open/short risk nếu connectivity hoặc shape interaction không được sửa đúng;
- congestion nặng hơn hoặc timing xấu hơn do route phải đi vòng.

## Limits / tool dependence
- Card này không định nghĩa numeric threshold cho pin-access quality.
- Cách tool tạo access points, chọn via candidate hoặc ưu tiên repair là implementation-specific. [Needs verification]
- PinAccess tốt không tự bảo đảm toàn design route-clean nếu congestion, timing hoặc signoff constraints khác chưa được giải quyết.

## Requires
- [[Pin]] — điểm kết nối vật lý mà router cần reach.
- [[LEF]] — mô tả pin shapes, obstructions và layer/rule context.
- [[RoutingGrid]] — không gian hợp lệ để đặt wire/via.
- [[Track]] · [[Pitch]] — quyết định local routing positions và density quanh pin.

## Used by
- [[DetailedRouting]] — kiểm tra và sửa feasibility tại từng pin.
- [[Routing]] — cần pin access hợp lệ để hoàn tất connectivity.
- [[DRC]] — pin access repair phải vẫn tôn trọng physical rules.
- [[Signoff]] — downstream checks có thể phơi bày lỗi liên quan routed connectivity/geometry. [Needs verification]

## Related
→ Pin geometry: [[Pin]] · [[LEF]]
→ Routing model: [[RoutingGrid]] · [[Track]] · [[Pitch]] · [[Via]]
→ Flow: [[Routing]] · [[DetailedRouting]] · [[Signoff]]
→ Constraints context: [[RoutingBlockage]] · [[PlacementDensity]] · [[CongestionAnalysis]] · [[DRC]]
