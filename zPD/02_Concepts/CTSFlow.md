---
tags: [concept, pnr-flow, cts]
group: PnR Flow
defined_in: ClockTreeSynthesis
used_by: [ClockTreeSynthesis, Routing, STA, Signoff]
requires: [ClockTreeSynthesis, CTSOptimization, ClockSkew, ClockLatency, Slew, DRVFixing]
chain: Chain_PnR_Flow
---
# CTSFlow

## Definition
**CTSFlow** là chuỗi thực thi nội tại bên trong [[ClockTreeSynthesis]] để xây dựng và tinh chỉnh physical clock tree theo từng pha có thứ tự.

Ở mức concept, luồng này đi qua bốn pha: **Clustering → Balancing → Routing Clock Tree → Post-Conditioning**.

## Position inside ClockTreeSynthesis
CTSFlow không phải một stage độc lập ngoài PnR flow, mà là “nội dung bên trong” bước [[ClockTreeSynthesis]].

Nó nhận bối cảnh từ placement/clock constraints của CTS, rồi chuyển clock từ mô hình cấu trúc ban đầu sang clock network đã có routing và được làm sạch chất lượng ở mức hậu định tuyến clock.

## Phase 1 — Clustering
Clustering nhóm các clock sinks theo độ gần vật lý và ý đồ clock-domain/skew-group, sau đó hình thành clock structure ban đầu để phân phối tải.

Ở pha này, tool thường phải đồng thời tôn trọng các ràng buộc fanout/capacitance/transition và các mục tiêu [[DRVFixing]] liên quan clock net ở mức khái niệm.

## Phase 2 — Balancing
Balancing điều chỉnh topology, phân bố delay, và vị trí/đặc tính buffer trên clock paths để giảm chênh lệch clock arrival time giữa sinks.

Pha này gắn trực tiếp với mục tiêu kiểm soát [[ClockSkew]] và [[ClockLatency]], đồng thời liên hệ với các chiến lược trong [[CTSOptimization]] (bao gồm cả controlled skew khi cần).

## Phase 3 — Routing Clock Tree
Sau khi topology và buffer placement đã đủ ổn định, clock tree được chuyển thành các clock nets vật lý có routing.

Trong một số flow, clock routing có thể đi kèm quy tắc đặc biệt như NDR, shielding, hoặc ưu tiên layer cao hơn; chi tiết phụ thuộc tool/PDK/flow cụ thể. [Needs verification]

Pha này là cầu nối trực tiếp sang [[Routing]] và ảnh hưởng chất lượng RC thực tế cho phân tích timing downstream.

## Phase 4 — Post-Conditioning
Post-Conditioning dùng hành vi clock thực tế hơn (sau khi có routing và/hoặc extraction phù hợp flow) để xử lý phần còn lại của transition/capacitance/fanout/skew/timing risk.

Pha này vẫn thuộc nội bộ CTSFlow và cần phân biệt với [[PostCTSOptimization]] ở giai đoạn downstream rộng hơn (deferred scope).

## What CTSFlow produces
Ở mức concept, CTSFlow tạo ra:
- Clock tree đã qua các bước hình thành, cân bằng, và routing clock ở mức nội bộ CTS.
- Bộ chất lượng clock thực tế hơn để handoff sang [[STA]] và các bước closure như [[Signoff]].
- Bối cảnh để tiếp tục tối ưu/fix các vấn đề clock-data interaction trong các stage sau.

## Trade-offs / limits
- Giảm skew/latency thường có trade-off với power/area và độ phức tạp implementation.
- Mức “aggressive” của balancing hay post-conditioning phụ thuộc mục tiêu QoR và budget của design.
- Các lựa chọn routing-policy cho clock nets (NDR/shielding/layer strategy) mang tính flow/tool/PDK-dependent. [Needs verification]

## Requires
- [[ClockTreeSynthesis]]
- [[CTSOptimization]]
- [[ClockSkew]]
- [[ClockLatency]]
- [[Slew]]
- [[DRVFixing]]

## Used by
- [[ClockTreeSynthesis]]
- [[Routing]]
- [[STA]]
- [[Signoff]]

## Related
- [[ClockTreeSynthesis]]
- [[CTSOptimization]]
- [[ClockSkew]]
- [[ClockLatency]]
- [[Slew]]
- [[DRVFixing]]
- [[Routing]]
- [[STA]]
- [[Signoff]]
- [[Chain_PnR_Flow]]
