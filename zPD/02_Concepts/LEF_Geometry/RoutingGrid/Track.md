---
tags: [concept, lef-geometry]
group: LEF Geometry — Routing Grid
defined_in: Tech LEF — TRACKS statement; stored in DEF post-floorplan
used_by: [RoutingGrid, Routing, CellAbstract]
requires: [Pitch, LEF]
chain: Chain_LEF_to_PnR
---
# Track

## Definition
Track là một đường thẳng ảo (imaginary line) trên [[RoutingGrid]], tại đó center của một wire hợp lệ có thể được đặt; có thể xem đây là legal routing line mà router được phép sử dụng. Tracks là discretization của không gian routing — thay vì Router có thể đặt wire tại bất kỳ tọa độ nào, nó chỉ được phép đặt wire tại các Track positions. Mỗi Track trên một layer cách Track liền kề đúng một Pitch. Số lượng Tracks available trên một layer trong một vùng diện tích nhất định là metric trực tiếp đánh giá routing capacity của vùng đó.

## Computed from
Tracks được define trong Tech LEF bằng TRACKS statement và stored vào DEF sau Floorplanning:

TRACKS Y 0.0 DO 200 STEP 0.2 LAYER M1 ;

Đọc là: trên một layer, tool instantiate các Tracks từ origin với STEP bằng [[Pitch]] của layer đó; preferred direction cụ thể của từng layer phụ thuộc công nghệ và rule deck.

Từ đây:
$$\text{Track}_{i} \text{ tại vị trí} = \text{Origin} + i \times \text{Pitch}, \quad i = 0, 1, 2, \ldots, N-1$$

$$\text{Số Tracks trong chiều cao H} = \left\lfloor \frac{H}{\text{Pitch}} \right\rfloor$$

**Cell height in Tracks**: Một số libraries mô tả chiều cao Standard Cell bằng số Tracks để phản ánh mức routing resource nội bộ của cell. Diễn giải chi tiết phụ thuộc library convention [Needs verification].

## Constrains
- **[[RoutingGrid]]**: tập hợp tất cả Tracks trên tất cả metal layers tạo thành Routing Grid; Track là đơn vị cơ bản của RoutingGrid
- **[[Routing]]**: Router bắt buộc phải đặt tất cả wires tại Track positions; off-track routing = DRC violation; Track availability tại một vùng quyết định xem Router có thể route qua vùng đó hay không (routing congestion = Tracks bị chiếm hết)
- **[[CellAbstract]]**: Pins trong Macro LEF phải có tâm align với Track positions — "Pin must land on Track grid"; nếu Pin off-track, Router phải tạo jog wire để tiếp cận, gây congestion và potential DRC violations

## Requires
- [[Pitch]] — Track spacing = Pitch; không thể define Tracks mà không biết Pitch của layer đó; Pitch là thông số vật lý cơ bản, Track là instantiation của Pitch spacing
- [[LEF]] — Tech LEF chứa TRACKS definitions cho tất cả metal layers; sau khi Floorplanning, TRACKS được copy vào DEF để PnR tool biết routing grid structure

## Used by
- [[RoutingGrid]] — tập hợp Tracks trên tất cả layers tạo thành RoutingGrid; Tracks là building blocks của RoutingGrid
- [[Routing]] — Router allocate Tracks cho từng net; một Track chỉ có thể được occupy bởi một wire tại mỗi vị trí (track assignment là resource allocation problem); routing congestion = tỷ lệ Tracks đã được occupied trên tổng Tracks available
- [[CellAbstract]] — Pin locations trong Macro LEF phải align với Track grid để Router có thể access Pin mà không cần off-track jogs; Track alignment là requirement khi library designer tạo cell layout

## Key insight
[USER REVIEW — draft suggestion]: Track là khái niệm chuyển đổi từ analog (không gian liên tục) sang digital (discrete positions) trong routing — và đây chính là yếu tố làm cho automated routing tractable. Nếu Router phải tìm wire path trong không gian liên tục, bài toán trở thành NP-hard không giải được trong thời gian hợp lý cho hàng tỷ nets. Bằng cách discretize thành Track positions, bài toán routing biến thành graph search problem có thể giải được. Điểm thực tế quan trọng: khi timing report cho thấy một path có Net Delay lớn bất thường, nguyên nhân thường là net đó bị detour dài vì Tracks bị congested tại một vùng — Router phải đi vòng → wire dài hơn → RC lớn hơn → Net Delay tăng. Đây là lý do congestion analysis sau Placement là bước quan trọng trước khi Routing.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formula: $\text{Track}_i = \text{Origin} + i \times \text{Pitch}$
→ Spacing between Tracks = [[Pitch]]
→ Defined in: [[LEF]] — Tech LEF TRACKS statement
→ Stored in: [[DEF]] — post-Floorplan DEF TRACKS section
→ Building block of: [[RoutingGrid]]
→ Consumed by: [[Routing]] · [[CellAbstract]] (Pin alignment)
→ Cùng nhóm: [[Pitch]] · [[RoutingGrid]] · [[Site]] · [[Row]] · [[PlacementGrid]]