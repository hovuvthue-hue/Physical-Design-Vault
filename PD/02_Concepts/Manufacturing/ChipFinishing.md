---
tags: [concept, manufacturing, pnr-flow, signoff]
group: Manufacturing
defined_in: Post-Route / Signoff Preparation
used_by: [Signoff, DFM]
requires: [Routing, PostRouteOptimization, DRC, AntennaEffect]
chain: Chain_PnR_Flow
---
# ChipFinishing

## Definition
ChipFinishing là lớp hoàn thiện vật lý muộn sau [[Routing]] và [[PostRouteOptimization]], trước khi design được đưa vào [[Signoff]] cuối cùng hoặc tape-out handoff. Mục tiêu của bước này là làm cho routed layout ổn định hơn về manufacturability, physical verification readiness và database/output readiness.

ChipFinishing không phải là một signoff deck hay manual sản xuất. Nó là umbrella concept cho các hoạt động hoàn thiện trên layout đã route, khi wire/via geometry đã đủ cụ thể để kiểm tra và cleanup theo các ràng buộc manufacturing/signoff. Chính sách cụ thể của từng foundry, PDK, process node, tool hoặc signoff flow cần được xác nhận riêng. [Needs verification]

## Position in flow
Trong PnR flow, ChipFinishing thường đứng sau các vòng sửa timing/DRV/route-quality chính của [[PostRouteOptimization]] và trước [[Signoff]]. Nếu layout còn thay đổi lớn sau finishing, các kiểm tra liên quan như [[DRC]], antenna, extraction hoặc timing có thể cần chạy lại tùy methodology. [Needs verification]

Ở mức concept, [[Routing]] tạo routed geometry; [[PostRouteOptimization]] sửa timing/DRV và route-quality còn lại; ChipFinishing chuẩn bị layout đó cho manufacturability-aware checks và final handoff.

## Main finishing activities
Các hoạt động dưới đây là nhóm concept thường nằm trong vùng ChipFinishing; việc bật/tắt, thứ tự chạy và tiêu chí pass/fail là tool/PDK/foundry-specific. [Needs verification]

### MetalFill
MetalFill là hoạt động thêm metal shapes để hỗ trợ density/manufacturing uniformity trên layout đã route. Fill có thể ảnh hưởng parasitic/coupling context, nên timing-aware hoặc signoff-aware fill policy cần được kiểm tra theo flow. [Needs verification]

Card này chỉ xem MetalFill như một hoạt động bên trong ChipFinishing, không tách thành concept card riêng.

### RedundantVia
RedundantVia là ý tưởng tăng robustness của via connection bằng cách dùng thêm via cut hoặc multi-cut via ở nơi có không gian hợp lệ. Mục tiêu concept là giảm rủi ro manufacturing/reliability tại điểm nối giữa các metal layer, nhưng lựa chọn vị trí, loại via và legality phụ thuộc rule deck và routing database. [Needs verification]

Card này chỉ xem RedundantVia như một hoạt động bên trong ChipFinishing, không tách thành concept card riêng.

### Antenna cleanup handoff
[[AntennaEffect]] có thể được xử lý trong routing, trong cleanup muộn hoặc trong vòng signoff feedback tùy flow. ChipFinishing là nơi tự nhiên để kiểm tra rằng các antenna-related fixes đã được handoff sang physical verification/signoff context, nhưng final antenna policy vẫn phụ thuộc methodology. [Needs verification]

### Final physical consistency
ChipFinishing cũng có thể bao gồm các cleanup vật lý cuối như kiểm tra DRC-oriented cleanup, đồng bộ filler/endcap/tap/decap consistency và chuẩn bị database/output cho downstream checks. Các mục này chỉ nên hiểu ở mức readiness concept, không phải danh sách command hoặc checklist tool.

## Relation to DFM
[[DFM]] là mục tiêu manufacturability-aware rộng hơn; ChipFinishing là một vùng hành động muộn trên layout đã route để hỗ trợ mục tiêu đó. Nói cách khác, DFM mô tả tiêu chí/chất lượng mong muốn, còn ChipFinishing gom các bước physical finishing có thể cải thiện readiness trước [[Signoff]].

ChipFinishing không bảo đảm yield và không thay thế DRC/DFM/signoff rule decks. Nó chỉ giảm rủi ro bằng cách đưa layout về trạng thái phù hợp hơn cho các kiểm tra downstream. [Needs verification]

## Relation to Signoff
[[Signoff]] xác nhận design bằng các phân tích và physical verification cuối cùng. ChipFinishing chuẩn bị dữ liệu layout trước khi các gate này chạy hoặc trước khi design được release sang signoff loop.

Nếu ChipFinishing thêm fill, đổi via, sửa antenna hoặc chỉnh geometry, các dữ liệu downstream như extraction, [[DRC]] và timing/SI context có thể cần được cập nhật lại tùy flow. [Needs verification]

## Limits / tool dependence
- ChipFinishing không định nghĩa density threshold, via rule, antenna ratio, command, report hoặc signoff pass/fail policy.
- MetalFill, RedundantVia và antenna cleanup là các hoạt động phụ thuộc foundry/PDK/tool/methodology. [Needs verification]
- ChipFinishing không thay thế [[DRC]], [[DFM]] hoặc [[Signoff]].
- Không nên dùng card này như manufacturing-tool manual.

## Requires
- [[Routing]] — cung cấp routed wire/via geometry để hoàn thiện.
- [[PostRouteOptimization]] — ổn định timing/DRV/route-quality trước khi finishing lớn.
- [[DRC]] — cung cấp physical-rule context cho cleanup và readiness.
- [[AntennaEffect]] — một nhóm rủi ro manufacturing có thể cần handoff/cleanup trước signoff.

## Used by
- [[Signoff]] — nhận layout đã finishing để chạy các gate cuối.
- [[DFM]] — dùng ChipFinishing như vùng hành động muộn cho manufacturability-aware improvements.

## Related
→ Flow: [[Routing]] · [[PostRouteOptimization]] · [[Signoff]]
→ Physical verification: [[DRC]] · [[AntennaEffect]]
→ Manufacturing quality: [[DFM]]
