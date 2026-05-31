---
tags: [concept, signoff, routing, reliability]
group: Signoff
defined_in: Signoff / Routing
used_by: [Routing, ParasiticExtraction, STA, Signoff]
requires: [InterconnectRC, SPEF, Routing, ParasiticExtraction]
chain: Chain_PnR_Flow
---
# SignalIntegrity

## Definition
SignalIntegrity là mức độ tín hiệu vẫn giữ được hành vi đúng dưới ảnh hưởng của physical interconnect effects. Trong routing/signoff context, SignalIntegrity tập trung vào việc routed wires, coupling, RC parasitics và switching lân cận có làm sai noise margin, delay hoặc timing behavior hay không.

SignalIntegrity là concept chất lượng/correctness của tín hiệu sau khi interconnect trở thành geometry vật lý, không phải một lệnh tool hoặc một report format.

## Crosstalk as SI issue
[[SignalIntegrity|Crosstalk]] là subtopic quan trọng của SignalIntegrity trong Routing. Khi hai routed nets gần nhau, coupling capacitance có thể làm switching trên aggressor net ảnh hưởng victim net.

Ở mức concept, Crosstalk có thể biểu hiện như:

- coupling-induced noise trên victim net;
- crosstalk-induced delay do switching tương quan giữa aggressor/victim;
- quan hệ aggressor/victim phụ thuộc topology, timing window và extracted parasitics. [Needs verification]

Card này không tạo card riêng cho Crosstalk.

## Timing and noise impact
SignalIntegrity liên quan tới [[STA]] vì coupling và extracted parasitics có thể thay đổi delay, [[Slack]] và [[Slew]]. Một route có connectivity đúng vẫn có thể cần xem xét SI nếu coupling giữa neighboring nets làm timing/noise behavior trở nên xấu.

Noise threshold, timing-window policy, glitch/noise propagation model và signoff methodology là tool/PDK/flow-specific. [Needs verification]

## Relation to Routing and ParasiticExtraction
[[Routing]] quyết định layer choice, spacing, parallel run length, detour và neighbor relationship giữa nets. Những quyết định này ảnh hưởng [[InterconnectRC]] và coupling mà [[ParasiticExtraction]] ghi vào [[SPEF]].

[[ParasiticExtraction]] là cầu nối từ routed geometry sang dữ liệu RC/coupling để [[STA]] và [[Signoff]] đánh giá SI/timing ở mức chính xác hơn pre-route estimate.

## Common mitigation concepts
Các hướng giảm SI risk thường được mô tả ở mức concept như:

- tăng spacing giữa nets nhạy;
- shielding bằng net tham chiếu phù hợp;
- đổi layer hoặc giảm đoạn chạy song song;
- routing detour để tách aggressor/victim;
- driver/sizing changes nếu timing/noise closure cần và flow cho phép. [Needs verification]

Mitigation policy, net criticality rule, shielding convention và acceptance criteria cần xác nhận theo project. [Needs verification]

## Limits / tool dependence
- Card này không định nghĩa SI threshold hoặc signoff pass/fail criterion.
- Không thay thế SI report, STA report hoặc signoff methodology.
- SI analysis phụ thuộc extracted parasitics, switching/timing context và tool model. [Needs verification]

## Requires
- [[InterconnectRC]] — physical RC/coupling model của routed nets.
- [[SPEF]] — exchange format thường dùng để đưa parasitics vào timing/SI analysis.
- [[Routing]] — tạo neighbor geometry và layer/spacing decisions.
- [[ParasiticExtraction]] — trích xuất parasitics từ routed layout.

## Used by
- [[Routing]] — cần SI awareness khi chọn layer/spacing cho nets nhạy. [Needs verification]
- [[ParasiticExtraction]] — cung cấp coupling/parasitic data cho SI/timing analysis.
- [[STA]] — dùng parasitics để đánh giá delay/slack/slew dưới context timing.
- [[Signoff]] — kiểm tra SI/timing/noise theo methodology của flow. [Needs verification]

## Related
→ Flow: [[Routing]] · [[ParasiticExtraction]] · [[STA]] · [[Signoff]]
→ Data/model: [[InterconnectRC]] · [[SPEF]]
→ Timing effects: [[Slack]] · [[Slew]]
→ Subtopic: [[SignalIntegrity|Crosstalk]]
