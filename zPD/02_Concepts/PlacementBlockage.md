---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Floorplanning step — used during and after MacroPlacement
used_by: [MacroPlacement, Placement, Floorplanning]
requires: [CoreArea, MacroPlacement]
chain: Chain_PnR_Flow
---
# PlacementBlockage

## Definition
Placement Blockage là vùng trong Core Area được đánh dấu cấm hoặc hạn chế Placement tool đặt Standard Cells/Macros vào (tùy loại). Placement Blockages được dùng chủ yếu để quản lý congestion hotspots quanh Macros và để dự trữ không gian cho routing channels, power straps, và các physical requirements khác.

Về bản chất, Placement Blockage là cơ chế điều chỉnh **legal placement availability** và placement density theo vùng: nó giảm demand placement (cell density/local utilization) tại các khu vực nhạy cảm, từ đó gián tiếp ảnh hưởng congestion posture trước CTS/Routing. Placement Blockage **không trực tiếp** quy định layer nào được route; phần đó thuộc phạm vi [[RoutingBlockage]].

**4 loại Placement Blockage:**

| Loại | Hành vi | Use case |
|---|---|---|
| **Hard** | Cấm hoàn toàn tất cả cell placement trong vùng | Vùng dự trữ cho routing channels, power straps |
| **Partial** | Cho phép placement đến một ngưỡng utilization xác định trước | Vùng routing-heavy nhưng vẫn cần một số logic |
| **Soft** | Chỉ cho phép một số cell types nhất định (Buffers, Inverters, Clock gaters) | Vùng chimneys giữa Macros — chỉ buffers được allowed |
| **Macro-only** | Cấm placement Macros trong vùng, nhưng Standard Cells vẫn được phép | Bảo vệ SC areas khỏi bị tool tự động đặt thêm Macros |

**Halo** — loại Placement Blockage đặc biệt gắn với Macro:
- Halo là một vùng keep-out có kích thước cố định bao quanh một Macro cụ thể
- Khi Macro di chuyển (trước khi được FIXED), Halo di chuyển cùng Macro
- Mục đích: đảm bảo có đủ không gian routing quanh Macro Pins và đủ space cho power rail tapping
- Kích thước Halo phụ thuộc kích thước Macro, mật độ Pin và nhu cầu routing/power quanh macro

**Phân biệt Placement Blockage vs Obstruction trong LEF:**

Obstruction (OBS trong CellAbstract) là vùng metal bên trong cell cấm external routing — đây là thuộc tính của cell LEF. Placement Blockage là constraint ở design-level do PD engineer tạo ra — nó định nghĩa vùng cấm placement trong Core Area. Hai khái niệm khác nhau hoàn toàn về nguồn gốc và scope.

## Computed from

Placement Blockages được tạo thủ công bởi PD engineer dựa trên phân tích congestion và chiến lược MacroPlacement:

**Khi nào cần Halo:**

Mỗi Macro cần Halo tối thiểu đủ để:
- Router có đủ Tracks để đi dây vào/ra Macro Pins trên mỗi cạnh
- Có không gian đặt Tap Cells (tapping VDD/VSS) giữa Macro và SC Rows lân cận
- Tránh Placement tool pack Standard Cells quá sát cạnh Macro → routing từ Macro Pins bị blocked

**Khi nào cần Hard Blockage:**

Đặt Hard Blockage tại những vùng routing-critical nơi Standard Cells tạo ra quá nhiều congestion: dọc theo chimneys giữa Macros stack, tại corners của Core Area gần IO Pads, dọc theo power strap corridors.

**Corner Blockage:**

Đôi khi chỉ cần đặt blockage nhỏ tại các góc của Macro hoặc tại corners của Core — các corners thường là congestion hotspots do routing wires từ nhiều hướng tập trung vào.

## Constrains
- **[[Placement]]**: Placement Blockages thu hẹp Placeable Standard Cell Area; Standard Cell Utilization sẽ cao hơn nếu nhiều Blockages được đặt
- **[[Routing]]**: Placement Blockage hỗ trợ routability theo hướng gián tiếp bằng cách ngăn Standard Cells lấp đầy vùng corridor; nó không phải layer-level routing rule

## Requires
- [[CoreArea]] — Placement Blockages được đặt bên trong Core Area
- [[MacroPlacement]] — Halo Blockages phụ thuộc vào Macro positions

## Used by
- [[MacroPlacement]] — Halos gắn trực tiếp với từng Macro
- [[Placement]] — Placement tool read Blockages trước khi place Standard Cells
- [[Floorplanning]] — Blockages là output của Floorplanning step

## Key insight
[USER REVIEW — draft suggestion]: Sử dụng Soft Blockage (chỉ cho phép Buffers) trong chimneys giữa Macros thay vì Hard Blockage là thực hành tốt — nó cho phép optimizer chèn Buffers để fix long-wire timing violations trong chính vùng đó, trong khi vẫn ngăn Placement engine lấp đầy bằng Standard Cells. Hard Blockage quá nhiều → Utilization trên báo cáo tăng ảo nhưng thực tế routing resource bị waste; quá ít → congestion hotspots xuất hiện sau Routing.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Types: Hard · Partial · Soft · Macro-only
→ Special: Halo (follows Macro movement, fixed size)
→ vs. [[Obstruction]]: Obstruction là trong CellAbstract LEF; PlacementBlockage là design-level constraint
→ Consumed by: [[Placement]] · [[MacroPlacement]] · [[Floorplanning]]
→ Cùng nhóm: [[RoutingBlockage]] · [[MacroPlacement]] · [[Floorplanning]]
