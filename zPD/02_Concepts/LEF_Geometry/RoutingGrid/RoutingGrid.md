---
tags: [concept, lef-geometry]
group: LEF Geometry — Routing Grid
defined_in: Tech LEF (Tracks per layer) + DEF (instantiated post-Floorplan)
used_by: [Routing, CellAbstract, Placement]
requires: [Track, Pitch, LEF, DEF]
chain: Chain_LEF_to_PnR
---
# RoutingGrid

## Definition
Routing Grid là toàn bộ mạng lưới các Tracks trên tất cả metal layers trong Core area, tạo thành không gian ba chiều (x, y, layer) mà Router sử dụng để đặt tất cả signal wires. Routing Grid là abstraction cho phép Router xem chip như một 3D graph — mỗi Track intersection là một node, mỗi Track segment giữa hai intersections là một edge, và Vias giữa các layers là vertical edges. Routing problem = graph shortest-path problem trên Routing Grid.

Routing Grid tách biệt với [[PlacementGrid]]: PlacementGrid quản lý legal vị trí đặt cell, còn RoutingGrid quản lý legal tài nguyên đi dây.

## Computed from
Routing Grid được tạo bằng cách instantiate Tracks cho mỗi metal layer theo thông tin từ Tech LEF. Mỗi metal layer có hướng ưu tiên (preferred routing direction) để minimize cross-layer coupling:

| Layer | Hướng ưu tiên | Ghi chú |
|---|---|---|
| Lower/upper layers | Preferred direction theo công nghệ | Router dùng để quản lý routing resource |
| Adjacent layers | Thường trực giao (orthogonal) | Giảm xung đột hướng đi dây trong Manhattan routing |

Mật độ Routing Grid tại một vùng được đo bằng **routing capacity** = tổng số Tracks available trên tất cả layers tại vùng đó. **Routing congestion** xảy ra khi routing demand vượt quá routing capacity — đây là tín hiệu rủi ro quan trọng cần check sau Placement.

$$\text{Routing Capacity}_{\text{region}} = \sum_{\text{layers}} \text{Tracks}_{layer} \times (1 - \text{Blockage Ratio}_{layer})$$

Một phần Routing Grid bị blocked bởi: PDN (power stripes chiếm một số Tracks), Clock Tree shielding wires, OBS trong Macro LEF, và Macro boundaries.

## Constrains
- **[[Routing]]**: Routing Grid là không gian duy nhất Router được phép hoạt động; mọi wire phải nằm trên Track, mọi via phải nằm tại Track intersection; Router cannot create new Tracks — chỉ allocate Tracks đã có
- **[[Placement]]**: Congestion-aware Placement phải ensure rằng cell density không quá cao tại bất kỳ vùng nào — cell density cao → ít Tracks còn trống → routing congestion; post-Placement congestion map là key quality metric trước Routing
- **[[CellAbstract]]**: Pin alignment với Routing Grid là design requirement cho Standard Cell library — Pins phải nằm tại Track intersections để Router có thể access trực tiếp bằng Via drop

## Requires
- [[Track]] — Routing Grid = tập hợp của tất cả Tracks trên tất cả layers; Track là building block atomic của Routing Grid
- [[Pitch]] — Pitch của từng layer xác định Track density trên layer đó; Pitch nhỏ hơn → nhiều Tracks hơn → Routing Grid dày đặc hơn
- [[LEF]] — Tech LEF define Tracks cho tất cả layers; Routing Grid không thể được built nếu thiếu Tech LEF
- [[DEF]] — post-Floorplan DEF instantiate Routing Grid thực tế cho design cụ thể; PDN routes trong DEF mark một số Tracks là blocked

## Used by
- [[Routing]] — Router là consumer chính của Routing Grid; routing algorithm = resource allocation problem trên Routing Grid; Global Routing assign nets to Track regions; Detailed Routing assign nets to specific Tracks
- [[CellAbstract]] — library designer phải ensure Pin locations align với Routing Grid khi tạo Macro LEF; misaligned Pins làm Router phải tạo off-grid jogs → congestion
- [[Placement]] — Placement tool estimate routing congestion bằng cách project net connections lên Routing Grid; vùng nào có Track utilization dự kiến > 90% thì Placement tool cần rearrange cells

## Key insight
[USER REVIEW — draft suggestion]: Routing Grid là nơi mà tất cả physical constraints hội tụ — LEF geometry rules (Pitch, Track direction), power planning (PDN blockages), và design hierarchy (Macro OBS) cùng nhau xác định routing feasibility. Một trong những skills quan trọng nhất của PD engineer là đọc congestion map sau Placement và identify hotspots trước khi chạy Routing — vì một congestion hotspot trong Routing Grid thường không thể được fix bởi Router (Router chỉ route những gì có thể route, không thể tạo thêm Tracks) mà phải được fix bằng cách thay đổi Floorplan hoặc Placement. Thực tế: ở advanced nodes như 7nm, M1 Routing Grid có Pitch ≈ 40nm và một chip lớn có thể có hàng tỷ Track segments — đây là lý do Routing runtime đo bằng giờ đến ngày, và tại sao Placement quality ảnh hưởng sâu sắc đến Routing convergence.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Composed of: [[Track]] × all metal layers
→ Track spacing = [[Pitch]] per layer
→ Defined in: [[LEF]] (Tech LEF) · [[DEF]] (instantiated post-Floorplan)
→ Must align with: [[PlacementGrid]] (key constraint trong Physical Design)
→ Consumed by: [[Routing]] · [[Placement]] (congestion estimation)
→ Cùng nhóm: [[Pitch]] · [[Track]] · [[Site]] · [[Row]] · [[PlacementGrid]]