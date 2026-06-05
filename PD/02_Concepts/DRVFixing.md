---
tags: [concept, pnr-flow, placement, timing]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [PreCTSOptimization, PostRouteOptimization, Placement, ClockTreeSynthesis, Signoff]
requires: [STA, Slack, Placement]
chain: Chain_PnR_Flow
---
# DRVFixing

## Definition
**DRVFixing** là hoạt động sửa các vi phạm design rule dạng điện (electrical DRV) được phát hiện trong các stage tối ưu của PnR, từ sau [[Placement]] đến post-route.

Mục tiêu là đưa design về trạng thái “khỏe điện” hơn trước khi đi tiếp sang CTS, Routing, hoặc các bước downstream.

## Common DRV types

Ba loại DRV phổ biến trong pre-CTS context, mỗi loại gắn với một loại pin khác nhau:

**max_capacitance** — constraint trên **output pin**
Giới hạn maximum load capacitance mà một output pin được phép drive. Defined trong cell library hoặc optionally set bởi designer qua SDC. Violation khi tổng capacitance tải quá lớn so với khả năng driver.
Fix methods: tăng driver strength (upsize cell), insert buffers để chia tải, net splitting nếu cần.

**max_transition** — constraint trên **input pin** (không phải output pin)
Định nghĩa maximum input slew (transition time) cho phép tại một pin. Violation khi driver quá yếu hoặc net load quá nặng → output slew thoải → input slew tại receiver vượt ngưỡng.
Fix methods: tăng driver strength, insert buffers để split long/high-capacitance nets và kiểm soát slew.

**max_fanout** — constraint trên **output pin**
Giới hạn số lượng loads tối đa mà một output pin có thể drive. Violation khi single driver feed quá nhiều gates → excessive capacitance → slow transitions.
Fix methods: insert buffers/repeaters để split fanout load.

Các rule cụ thể và ngưỡng phụ thuộc library/tool/flow [Needs verification].

## Why DRV fixing matters before CTS and after route
Nếu để tồn tại electrical DRV trước CTS:
- timing closure có thể khó hoặc kém ổn định hơn,
- QoR downstream có thể xấu đi,
- effort sửa lỗi ở các stage sau có thể tốn kém hơn.

Vì vậy DRVFixing thường là một phần quan trọng của pre-CTS cleanup. Sau [[Routing]], [[PostRouteOptimization]] tiếp tục dùng kết quả [[STA]] với extracted parasitics để sửa DRV còn lại trong bối cảnh [[Slew]], capacitance, và fanout thực tế hơn.

## Typical conceptual fixes
Ở mức khái niệm, DRVFixing có thể bao gồm:
- điều chỉnh/bổ sung buffer hợp lý,
- điều chỉnh cell strength (upsize/downsize tùy mục tiêu),
- tái cân bằng tải để giảm stress điện trên net.

Chiến lược chính xác cần bám STA context và ràng buộc implementation thực tế [Needs verification].

## Requires
- [[STA]]
- [[Slack]]
- [[Placement]]

## Used by
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[Placement]]
- [[ClockTreeSynthesis]]
- [[Signoff]]

## Related
- [[STA]]
- [[Slack]]
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[Placement]]
- [[ClockTreeSynthesis]]
