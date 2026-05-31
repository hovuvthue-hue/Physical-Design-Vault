---
tags: [concept, signoff, routing, physical-verification]
group: Signoff
defined_in: Signoff / Physical Verification
used_by: [DetailedRouting, Routing, Signoff]
requires: [LEF, DEF, RoutingGrid]
chain: Chain_PnR_Flow
---
# DRC

## Definition
DRC (Design Rule Check) là kiểm tra layout geometry so với các manufacturing/technology rules. Trong Physical Design, DRC trả lời câu hỏi: các metal shapes, spacing, width, via và enclosure có phù hợp với rule context của process hay không.

DRC là concept về geometry correctness, không phải concept về logic intent. Rule deck, signoff deck, cách phân loại violation và policy pass/fail phụ thuộc foundry/PDK/tool/signoff flow. [Needs verification]

## Relation to Routing
Trong [[Routing]], đặc biệt là [[DetailedRouting]], router phải tạo wire và [[Via]] geometry theo [[RoutingGrid]], layer rules và obstruction/blockage context để tránh violation ngay từ lúc route. Vì vậy routing-stage cleanup thường cố giảm lỗi DRC trước khi handoff sang downstream checks.

Tuy nhiên DRC cleanup trong Routing không đồng nghĩa với final signoff DRC. Final [[Signoff]] có thể dùng rule deck, extraction/layout view hoặc signoff methodology khác với in-route checker; mức tương quan giữa hai môi trường là tool/PDK/flow-specific. [Needs verification]

## What DRC checks conceptually
Ở mức concept, DRC thường bao phủ các nhóm lỗi hình học như:

- spacing giữa các shapes hoặc nets lân cận;
- width của wire/shape trên từng layer;
- via, cut, enclosure và overlap giữa via với metal;
- tương tác với obstruction, blockage hoặc macro boundary;
- một số kiểm tra manufacturing-related khác tùy rule deck. [Needs verification]

Các ví dụ này chỉ mô tả loại quan hệ hình học; card này không định nghĩa rule values hoặc foundry thresholds.

## DRC vs LVS
DRC kiểm tra layout geometry so với manufacturing/technology rules. LVS kiểm tra layout-vs-schematic/netlist connectivity equivalence, tức layout có biểu diễn đúng connectivity logic mong muốn hay không.

Hai loại check có thể liên quan trong debug, nhưng chúng trả lời hai câu hỏi khác nhau: DRC là rule/geometry correctness, còn LVS là connectivity equivalence. Card này không tạo card riêng cho LVS.

## Relation to AntennaEffect
[[AntennaEffect]] có thể được kiểm tra hoặc báo cùng nhóm physical verification trong một số flow, nhưng nó là rủi ro manufacturing liên quan charge accumulation trong fabrication chứ không chỉ là spacing/width/enclosure geometry thông thường. Antenna rule ratios, repair policy và signoff handling là foundry/PDK/tool-specific. [Needs verification]

## Limits / tool dependence
- DRC card này không thay thế signoff manual, runset hoặc rule deck.
- Tên rule, severity, waiver policy và auto-fix strategy là project-specific. [Needs verification]
- DRC-clean ở một bước trung gian không bảo đảm final signoff clean nếu layout tiếp tục thay đổi hoặc môi trường check khác nhau. [Needs verification]

## Requires
- [[LEF]] — cung cấp abstract layer, pin, obstruction và routing-rule context cho implementation.
- [[DEF]] — chứa placed/routed design geometry ở các mốc trao đổi trong flow.
- [[RoutingGrid]] — mô tả không gian/track hợp lệ mà router dùng để đặt wire/via geometry.

## Used by
- [[DetailedRouting]] — tạo và sửa routed geometry theo physical-rule constraints.
- [[Routing]] — cần DRC-aware routing để giảm lỗi trước extraction/signoff.
- [[Signoff]] — dùng DRC như một physical verification gate trước tape-out.

## Related
→ Flow: [[Routing]] · [[DetailedRouting]] · [[Signoff]]
→ Geometry context: [[LEF]] · [[DEF]] · [[RoutingGrid]] · [[Via]]
→ Manufacturing-related: [[AntennaEffect]]
