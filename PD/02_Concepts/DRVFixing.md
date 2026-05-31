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
Trong phạm vi concept-level pre-CTS, DRV thường xoay quanh:
- **max transition / slew violation**
- **max capacitance violation**
- **max fanout violation**

Các rule cụ thể và ngưỡng cụ thể phụ thuộc library/tool/flow [Needs verification].

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
