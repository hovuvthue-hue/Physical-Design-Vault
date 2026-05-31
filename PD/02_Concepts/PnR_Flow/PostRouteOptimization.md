---
tags: [concept, pnr-flow, routing, optimization]
group: PnR Flow
defined_in: Routing / Post-Route
used_by: [STA, Signoff, Routing]
requires: [Routing, ParasiticExtraction, SPEF, STA, Slack, DRVFixing, InterconnectRC]
chain: Chain_PnR_Flow
---
# PostRouteOptimization

## Definition
**PostRouteOptimization** là giai đoạn tối ưu hóa sau [[Routing]], [[DetailedRouting]], và [[ParasiticExtraction]], khi routed wire/via geometry đã có và parasitics đã được extract để đưa vào phân tích timing.

Khác với các bước tối ưu sớm hơn, giai đoạn này làm việc trong bối cảnh hậu route: [[InterconnectRC]] không còn chỉ là ước lượng từ placement/topology dự đoán, mà được lấy từ geometry thực tế và trao đổi qua [[SPEF]] cho [[STA]]. Vì vậy kết quả [[Slack]], [[Slew]], và các DRV điện phản ánh gần hơn trạng thái layout đã route.

## Position in flow
Trong flow PnR, PostRouteOptimization nằm sau Routing/ParasiticExtraction và trước [[Signoff]] cuối cùng. Nó là lớp cleanup post-route nhằm giảm rủi ro timing/DRV còn lại trước khi design bước vào các kiểm tra signoff nghiêm ngặt hơn.

PostRouteOptimization không phải là final Signoff. Nó sử dụng kết quả phân tích hậu route để sửa các vấn đề còn lại, còn Signoff là bước xác nhận độc lập hơn rằng design đã đáp ứng các tiêu chí closure cần thiết. Tiêu chí handoff giữa PostRouteOptimization và Signoff phụ thuộc flow/signoff policy cụ thể. [Needs verification]

## Why actual parasitics matter
Trước Routing, RC của net thường được estimate từ vị trí cell, wirelength dự đoán, congestion model, hoặc topology giả định. Các estimate này hữu ích để guide placement, CTS, và optimization sớm, nhưng chưa biết chính xác layer assignment, detour, spacing, và via usage cuối cùng.

Sau Routing, wire và [[Via]] geometry thực tế tạo ra [[InterconnectRC]] cụ thể hơn. [[ParasiticExtraction]] biến geometry đó thành extracted parasitics, [[SPEF]] mang dữ liệu này vào [[STA]], và post-route STA cập nhật [[NetDelay]], [[Slew]], cũng như [[Slack]] setup/hold. Đây là lý do một path đã ổn trước route vẫn có thể cần cleanup sau khi SPEF được annotate.

Nếu route hoặc ECO-like change làm thay đổi geometry đã route, extraction/timing trước đó có thể không còn đại diện cho layout mới và cần được phân tích lại. Mức độ re-extract, incremental update, và điều kiện chấp nhận phụ thuộc tool/flow cụ thể. [Needs verification]

## Difference from PreCTSOptimization and PostCTSOptimization
- **Khác [[PreCTSOptimization]]**: PreCTSOptimization diễn ra sau placement và trước CTS, khi clock và RC vẫn còn nhiều phần ước lượng; PostRouteOptimization diễn ra sau route với extracted parasitics.
- **Khác [[PostCTSOptimization]]**: PostCTSOptimization làm việc sau CTS với propagated clock nhưng trước final routed parasitics; PostRouteOptimization dùng bối cảnh wire/via geometry đã route và SPEF.
- **Khác in-route cleanup / detailed routing repair**: in-route cleanup tập trung vào hoàn tất hoặc sửa route trong quá trình router hoạt động; PostRouteOptimization là lớp tối ưu sau khi có route/extraction/timing context rõ hơn.
- **Khác [[Signoff]]**: Signoff xác nhận closure; PostRouteOptimization áp dụng các chỉnh sửa có kiểm soát để tiến tới trạng thái đủ sạch cho Signoff.

## Main cleanup targets
Các mục tiêu thường gặp ở mức concept gồm:
- setup recovery dựa trên post-route [[STA]] và [[Slack]],
- hold recovery trên các fast path lộ ra sau routed RC/clock context,
- [[DRVFixing]] cho max transition/slew, max capacitance, hoặc fanout-related electrical issues,
- transition/[[Slew]] cleanup để giảm rủi ro delay lan truyền sang stage kế tiếp,
- capacitance/fanout cleanup trên các net/cell chịu tải lớn,
- localized buffering/sizing nếu flow cho phép,
- route hoặc ECO-like repair giới hạn khi cần chỉnh tác động của geometry,
- [[PowerOptimization|power/area recovery]] chỉ khi timing, DRV, và risk margin đang ở trạng thái chấp nhận được.

Thứ tự ưu tiên giữa setup, hold, DRV, SI, và recovery là tool/flow/project-specific. [Needs verification]

## Setup / hold / DRV trade-offs
Post-route cleanup có nhiều trade-off chéo:
- sửa hold bằng cách tăng delay có thể làm xấu setup ở path liên quan,
- cải thiện setup bằng sizing/buffering có thể tăng power, area, input load, và routing demand,
- sửa slew/cap có thể thay đổi [[NetDelay]], [[CellDelay]], và [[Slack]],
- route change có thể thay đổi [[InterconnectRC]], làm các kết quả extraction/STA cũ cần được cập nhật,
- một chỉnh sửa cục bộ có thể tạo regression ở mode/corner khác nếu không được kiểm tra đồng thời. [Needs verification]

Vì vậy PostRouteOptimization thường được xem như optimization incremental, localized, và minimal-impact thay vì một lần rewrite toàn bộ routed design.

## Power and area recovery
Sau các bước timing-driven optimization, design có thể còn cell mạnh hơn cần thiết, buffer dư, hoặc lựa chọn implementation tiêu thụ power/area cao hơn mức cần thiết. [[PowerAnalysis]] trong bối cảnh routed/extracted giúp nhận diện nơi có thể recovery.

Tuy nhiên power/area recovery sau route chỉ nên thực hiện khi timing và DRV đủ ổn định; nếu recovery làm mất margin, thay đổi [[Slew]], hoặc làm xấu setup/hold, design phải được phân tích lại. Chính sách recovery, thứ tự ECO, và tiêu chí dừng là flow-specific. [Needs verification]

## Limits / tool dependence
PostRouteOptimization là concept cấp flow, không phải danh sách command. Các chi tiết như optimizer pass order, acceptance criteria, ECO strategy, incremental extraction, SI-aware timing, và signoff correlation phụ thuộc tool, PDK, foundry deck, process node, và methodology nội bộ. [Needs verification]

Card này không định nghĩa TimingClosure hay PowerAreaRecovery như concept riêng; chúng được xem là các nhóm hoạt động bên trong post-route optimization context.

## Requires
- [[Routing]]
- [[DetailedRouting]]
- [[ParasiticExtraction]]
- [[SPEF]]
- [[InterconnectRC]]
- [[STA]]
- [[Slack]]
- [[Slew]]
- [[DRVFixing]]

## Used by
- [[STA]]
- [[Signoff]]
- [[Routing]]
- [[PowerOptimization]]
- [[PowerAnalysis]]

## Related
- [[PreCTSOptimization]]
- [[PostCTSOptimization]]
- [[Routing]]
- [[DetailedRouting]]
- [[ParasiticExtraction]]
- [[SPEF]]
- [[InterconnectRC]]
- [[STA]]
- [[Slack]]
- [[Slew]]
- [[DRVFixing]]
- [[PowerOptimization]]
- [[PowerAnalysis]]
- [[Signoff]]
