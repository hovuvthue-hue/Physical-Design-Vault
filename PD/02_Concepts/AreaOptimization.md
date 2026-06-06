---
tags: [concept, pnr-flow, placement, optimization]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [PreCTSOptimization, Placement, Signoff]
requires: [Placement, STA, Slack, StandardCell]
chain: Chain_PnR_Flow
---
# AreaOptimization

## Definition
**AreaOptimization** là giai đoạn tối ưu diện tích trong post-placement optimization, thực hiện sau timing optimization. Tool thực hiện downsizing cells xuống variants nhỏ hơn và xóa các buffers hoặc inverters không cần thiết để giảm area, với ràng buộc cứng là **chỉ target positive slack nets** — đảm bảo không có degradation timing và không tạo thêm DRV violations mới.

## What AreaOptimization does

**Cell Downsizing (thu hẹp drive strength):** Thay cell hiện tại bằng variant nhỏ hơn cùng logic function. Ví dụ: INV8X → INV4X. Điều kiện: net được target phải có đủ positive slack để absorb delay tăng nhẹ do driver yếu hơn, đồng thời không vi phạm max transition hoặc max capacitance constraints.

**Buffer/Inverter Removal (xóa buffers/inverters thừa):** Xóa các buffer hoặc inverter dư thừa đã được insert trong synthesis hoặc timing optimization trước đó, khi slack cho phép.

## Timing safety mechanism

AreaOptimization enforce tính an toàn qua cơ chế:
- Chỉ can thiệp vào nets có **positive slack** → delay tăng nhẹ sau downsizing vẫn nằm trong budget.
- Không được tạo thêm DRV violations mới (max transition, max capacitance, max fanout).
- Không được làm bất kỳ path nào chuyển từ positive sang negative slack.

## Position in flow

AreaOptimization diễn ra **tự động sau timing optimization theo mặc định**, nhưng cũng có thể được chạy tường minh khi cần. Trong post-placement flow, thứ tự điển hình là: timing optimization → area optimization → power optimization. Trong một số flows, area và power optimization được gộp trong cùng một pass.

## Relation to PowerOptimization

AreaOptimization và [[PowerOptimization]] chia sẻ kỹ thuật downsizing và buffer removal nhưng có primary objective khác nhau. AreaOptimization tập trung minimize diện tích cell; việc giảm dynamic power là phụ phẩm (cell nhỏ hơn → input capacitance nhỏ hơn → switching power giảm). [[PowerOptimization]] có phạm vi rộng hơn: bao gồm VT swapping để giảm leakage, adjust placement để shorten wirelength, và tuân thủ UPF power budgets. Cả hai đều bị ràng buộc chặt bởi timing và DRV context — không được compromise [[Slack]] để đổi lấy area hoặc power.

## Requires
- [[Placement]]
- [[STA]]
- [[Slack]]
- [[StandardCell]]

## Used by
- [[PreCTSOptimization]] — AreaOptimization là một phần của post-placement optimization loop trước CTS.
- [[Placement]] — area và utilization sau optimization ảnh hưởng đến placement legality và density.
- [[Signoff]] — area metrics sau optimization là inputs cho signoff area reporting.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Target condition: chỉ positive slack nets; không tạo DRV violations mới.
→ Techniques: Cell Downsizing · Buffer/Inverter Removal.
→ Closely related: [[PowerOptimization]] · [[PreCTSOptimization]] · [[DRVFixing]] · [[Slack]]
→ Cùng nhóm: [[PowerOptimization]] · [[PreCTSOptimization]] · [[MBFF]] · [[Placement]]