---
tags: [concept, pnr-flow, placement, power]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [PreCTSOptimization, PostRouteOptimization, Placement, Signoff]
requires: [PowerAnalysis, LeakagePower, Placement, STA]
chain: Chain_PnR_Flow
---
# PowerOptimization

## Definition
## Definition
**PowerOptimization** là nhóm hoạt động tối ưu nhằm giảm power tiêu thụ mà không compromising timing. Khi Power Optimization được enable, tool thực hiện các actions sau:

- Swap cells sang lower-power equivalents để giảm leakage.
- Dùng cells với drive strength nhỏ hơn để minimize area và power.
- Select higher-Vt cells để giảm thêm leakage power.
- Remove unnecessary buffers hoặc inverters để giảm dynamic power và area.
- Adjust cell placement để optimize wirelength cho dynamic power savings.
- Tuân thủ design constraints như UPF power budgets.

Trong ngữ cảnh post-placement (pre-CTS), đây là bước chạy sau timing optimization. Trong bối cảnh post-route, PowerOptimization có thể xuất hiện như power/area recovery bên trong [[PostRouteOptimization]], nhưng chỉ khi timing, DRV, và rủi ro signoff còn được kiểm soát.
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

## Leakage optimization context
Trong context [[LeakagePower]], các hướng tối ưu phổ biến gồm:
- **Multi-Vt cell swap**: đổi sang HVT có thể giảm leakage, nhưng có thể làm path chậm hơn.
- **LVT/SVT/HVT trade-off**: LVT thường ưu tiên timing hơn leakage; HVT thường ưu tiên leakage hơn timing; SVT thường nằm giữa hai hướng này. Mức độ trade-off phụ thuộc library và corner. [Needs verification]
- **Downsizing**: giảm drive strength có thể giảm leakage và area nếu [[Slack]], load/slew và DRV constraints vẫn cho phép.
- **Cell swap for lower leakage**: phải bị ràng buộc bởi [[STA]], [[Slack]], DRV, placement legality, pin access và legal/library equivalence. [Needs verification]
- **Body bias và logic-state stacking**: là cơ chế phụ thuộc device, architecture, PDK/methodology và trạng thái logic; không nên xem như generic placement levers trong mọi flow. [Needs verification]

## Requires
- [[PowerAnalysis]]
- [[LeakagePower]]
- [[Placement]]
- [[STA]]

## Used by
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[Placement]]
- [[Signoff]]

## Related
- [[PowerAnalysis]]
- [[LeakagePower]]
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[STA]]
- [[Signoff]]
