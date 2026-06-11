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
CTS tool nhóm các clock sinks (Flip-flops) liên quan thành clusters trước khi xây dựng clock tree, dựa trên vị trí vật lý, fanout, capacitance, và DRV constraints. Mỗi cluster đại diện cho một nhóm sequential cells cùng clock source. Clock tree được xây dựng theo DRV constraints (max capacitance, max fanout, max transition). Reports sau clustering cho thấy số lượng buffers, inverters, và gated clock cells được insert.

Ba chiến lược clustering:
- **Top-Down Clustering**: bắt đầu với một cluster lớn chứa toàn bộ sinks, sau đó đệ quy chia nhỏ thành các clusters nhỏ hơn.
- **Bottom-Up Clustering**: bắt đầu từ từng sink riêng lẻ và dần gộp thành clusters lớn hơn.
- **Iterative Clustering**: kết hợp cả top-down và bottom-up theo vòng lặp để cải thiện chất lượng clock tree.

## Phase 2 — Balancing
Balancing điều chỉnh clock paths để equalize clock arrival times và minimize clock skew. CTS tool cân bằng skew theo skew-group constraints; optimization được lặp lại cho đến khi skew targets đạt được cho tất cả skew groups hoặc không còn cải thiện thêm được nữa. CTS engine có thể: insert thêm buffers, modify clock tree topology, adjust clock branch depth, và apply useful skew optimization.

Pha này gắn trực tiếp với mục tiêu kiểm soát [[ClockSkew]] và [[ClockLatency]], đồng thời liên hệ với các chiến lược trong [[CTSOptimization]].

## Phase 3 — Routing Clock Tree
Clock nets được route vật lý theo dedicated routing rules, tuân theo NDR rules đã được define trước. Clock nets thường được route trên các metal layers cao hơn để: giảm resistance và capacitance, minimize coupling noise, cải thiện signal integrity, và giảm clock latency variation.

Routing rules thường được áp dụng phân biệt cho: trunk nets, leaf nets, và top-level clock routes. Multi-cut vias thường được dùng để cải thiện reliability, electromigration robustness, và via resistance.

Pha này là cầu nối trực tiếp sang [[Routing]] và ảnh hưởng chất lượng RC thực tế cho phân tích timing downstream.

## Phase 4 — Post-Conditioning
Giai đoạn tinh chỉnh CTS cuối cùng, nơi propagated clocks được dùng để repair các timing và DRV violations phát sinh do routed clock delays và extracted parasitics. Clock latency, transition, và skew thực tế được đánh giá sau clock routing; clock paths được tinh chỉnh để cải thiện skew, insertion delay, transition quality, và clock integrity tổng thể.

Pha này sửa các violations bao gồm: setup/hold/transition/capacitance/fanout. Routed clock nets có thể introduce: routing parasitics, additional RC delay, coupling effects, và clock latency variation. CPPR adjustment được áp dụng để cải thiện timing accuracy. Các hoạt động tinh chỉnh có thể bao gồm: buffer insertion, buffer resizing, useful skew optimization, và hold fixing.

Pha này vẫn thuộc nội bộ CTSFlow và cần phân biệt với [[PostCTSOptimization]] ở giai đoạn downstream rộng hơn.

## Recommended Three-Stage Setup Approach

Ngoài 4 internal execution phases, CTS còn có một recommended workflow để set up và validate clock tree trên design mới, dùng ba `cts_balance_mode` settings theo thứ tự leo thang:

**Stage 1 — Cluster:**
Tool nhóm tất cả clock sink pins thành clusters và chỉ insert buffers để thỏa mãn DRV constraints (transition time, capacitance, fanout limits). Không có skew balancing, không có timing optimization, không có clock net routing. Mục tiêu: xác định xem maximum insertion delay path có hợp lý không — tìm transition violations từ floorplan issues (blockages, narrow channels), FIXED clock gates, divider flops, power domain crossings, và uneven cluster distribution across skew groups.

Trong Clock Tree Debugger, DRV-inserted buffers phân biệt được với balancing buffers; insertion delay bar chart highlight maximum delay path.

**Stage 2 — Trial:**
Tool chạy full clustering stage cộng virtual delay balancing — thay vì insert real balancing buffers, tool áp dụng estimated virtual delays để simulate một balanced clock tree. Clock nets vẫn chưa được route. Đây là fast và low-cost run. Mục tiêu: xác định liệu balancing constraints có được setup đúng không — tìm large skew hoặc insertion delay tăng bất thường từ conflicting skew group constraints.

Trong CTD, virtual delay values được annotate trực tiếp trên affected buffer nodes; arrival time chart hiển thị simulated sink arrivals so với balancing target line, cho thấy liệu skew group constraints có achievable không.

**Stage 3 — Full:**
Tool chạy complete CTS flow: sink clustering, real buffer và inverter insertion cho DRV compliance và skew balancing, clock net routing qua NanoRoute với NDR rules, và concurrent data path timing optimization. Đây là production mode và là default. Clock insertion delay tại stage này là real và propagated.

Trong CTD, balancing buffers phân biệt với DRV buffers theo màu; arrival time chart hiển thị actual propagated clock arrivals với minimized skew; NDR routing confirmation xuất hiện trong tree header.

Workflow thực tế: chạy cluster trước để kiểm tra floorplan và DRV, sau đó trial để validate constraints trước khi commit vào full run — tiết kiệm thời gian bằng cách phát hiện vấn đề sớm ở các stage ít tốn kém hơn.

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
