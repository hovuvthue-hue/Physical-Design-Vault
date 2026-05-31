---
tags: [concept, pnr-flow, placement, power]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [PreCTSOptimization, PostRouteOptimization, Placement, Signoff]
requires: [PowerAnalysis, Placement, STA]
chain: Chain_PnR_Flow
---
# PowerOptimization

## Definition
**PowerOptimization** là nhóm hoạt động tối ưu sau placement nhằm giảm power tiêu thụ, đồng thời vẫn giữ các ràng buộc timing và tính hợp lệ vật lý của layout.

Trong ngữ cảnh L8, đây là phần của post-placement optimization trước [[ClockTreeSynthesis]]. Trong bối cảnh post-route, PowerOptimization có thể xuất hiện như power/area recovery bên trong [[PostRouteOptimization]], nhưng chỉ khi timing, DRV, và rủi ro signoff còn được kiểm soát.

## Analysis vs optimization
- [[PowerAnalysis]]: đo/ước lượng và phân rã power để hiểu nguyên nhân.
- **PowerOptimization**: thay đổi lựa chọn implementation (cell/placement/post-placement tuning) để cải thiện kết quả power.

Nói ngắn gọn: analysis cho biết “vấn đề ở đâu”, optimization xử lý “cách sửa vấn đề đó”.

## Common optimization levers
Các đòn bẩy thường được đề cập ở mức concept:
- **Cell sizing**: điều chỉnh drive strength theo nhu cầu thực tế.
- **Vt choice / cell swap**: cân bằng giữa leakage và timing.
- **Capacitance/activity-related reduction**: giảm tải/hoạt tính chuyển mạch hiệu dụng thông qua lựa chọn implementation phù hợp.
- **Area trade-off**: một số hướng giảm power có thể đổi lấy area hoặc ngược lại.

Chi tiết hành vi phụ thuộc library, tool và flow setup cụ thể [Needs verification].

## Trade-offs
PowerOptimization là bài toán multi-objective:
- giảm leakage có thể làm timing khó hơn,
- giảm dynamic có thể ảnh hưởng area hoặc routability,
- ưu tiên timing gắt có thể đẩy power tăng.

Do đó không nên giả định PowerOptimization luôn cải thiện đồng thời mọi chỉ số PPA trong mọi design. Sau route, recovery có thể làm thay đổi load, [[Slew]], hoặc margin timing, nên cần được ràng buộc bởi STA/signoff context và policy cụ thể [Needs verification].

## Requires
- [[PowerAnalysis]]
- [[Placement]]
- [[STA]]

## Used by
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[Placement]]
- [[Signoff]]

## Related
- [[PowerAnalysis]]
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[STA]]
- [[Signoff]]
