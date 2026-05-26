---
tags: [concept, lef-geometry]
group: LEF Geometry — Routing Grid
defined_in: Tech LEF — LAYER section, keyword PITCH
used_by: [Track, RoutingGrid, Routing, ParasiticExtraction]
requires: [LEF]
chain: Chain_LEF_to_PnR
---
# Pitch

## Definition
Pitch là khoảng cách từ tâm đến tâm giữa hai dây dẫn (wire) song song liền kề trên cùng một metal layer. Pitch là thông số nền tảng nhất của Routing Grid — nó xác định mật độ tối đa của các wires có thể được đặt trên một metal layer mà không vi phạm DRC spacing rules. Pitch được Foundry định nghĩa cho từng metal layer trong Tech LEF và là hằng số cố định cho một process node — PD engineer không thể thay đổi giá trị này.

## Computed from
$$\text{Pitch} = \text{Min Wire Width} + \text{Min Wire Spacing}$$

Trong đó:
- **Min Wire Width**: chiều rộng tối thiểu của dây dẫn trên layer đó (đủ để dòng điện chạy qua mà không bị electromigration, đủ để lithography in được)
- **Min Wire Spacing**: khoảng cách tối thiểu giữa hai cạnh ngoài của hai wires liền kề (đủ để tránh rò điện và đảm bảo process yield)

Ví dụ từ slide 17 của tài liệu: `TRACKS Y 0.0 DO 200 STEP 0.2 LAYER M1` — STEP 0.2 chính là Pitch của M1 = 0.2um. Tương tự `TRACKS X 0.0 DO 100 STEP 0.2 LAYER M2` — Pitch M2 = 0.2um.

Pitch có xu hướng giảm theo mỗi technology node (scaling): TSMC 22nm M1 Pitch ≈ 0.1um, TSMC 7nm M1 Pitch ≈ 0.04um. Đây là một trong những metrics chính đánh giá mức độ scaling của một process node.

## Constrains
- **[[Track]]**: mỗi Track là một đường thẳng trên Routing Grid; khoảng cách giữa các Tracks liên tiếp trên cùng một layer = Pitch; số lượng Tracks trên một đoạn chiều dài L = L / Pitch
- **[[RoutingGrid]]**: toàn bộ Routing Grid được xây dựng bằng cách đặt các Tracks cách nhau đúng một Pitch; Pitch nhỏ hơn → nhiều Tracks hơn trên cùng diện tích → routing density cao hơn
- **[[Site]]**: Cell height của Standard Cell thường được express theo số Tracks: "8-track cell" nghĩa là chiều cao cell = 8 × M1 Pitch; đây là cách Pitch kết nối LEF Geometry group Routing với Placement sub-group

## Requires
- [[LEF]] — Pitch được define trong Tech LEF tại LAYER section với keyword PITCH; không có Tech LEF không thể biết Pitch của bất kỳ layer nào; Pitch là thuộc tính của technology, không phải của design

## Used by
- [[Track]] — Pitch định nghĩa khoảng cách giữa các Tracks; Track không tồn tại độc lập mà là instantiation của Pitch spacing trên một cụ thể metal layer
- [[RoutingGrid]] — Routing Grid = tập hợp tất cả Tracks trên tất cả layers; Pitch của mỗi layer quyết định density của grid trên layer đó
- [[Routing]] — Router chỉ được đặt wires tại các Track positions (bội số của Pitch từ origin); off-pitch routing gây DRC violations
- [[ParasiticExtraction]] — Pitch (đặc biệt spacing component) ảnh hưởng trực tiếp đến Coupling Capacitance giữa adjacent wires: $C_{coupling} \propto 1/\text{Spacing}$; tại advanced nodes, Pitch nhỏ dẫn đến Spacing nhỏ → Coupling C tăng vọt

## Key insight
[USER REVIEW — draft suggestion]: Pitch là thông số đơn giản nhất nhưng có tác động lan rộng nhất trong Physical Design — nó đồng thời quyết định routing density (performance), wire resistance per unit length (timing), và coupling capacitance với neighboring wires (signal integrity). Sự thu nhỏ Pitch theo mỗi technology node là động lực chính của Moore's Law: Pitch nhỏ hơn → nhiều wires trên cùng diện tích → nhiều transistors có thể connect → chip mạnh hơn trong cùng area. Nhưng trade-off là Coupling Capacitance tăng theo $1/\text{Spacing}$ — ở TSMC 3nm, Coupling C chiếm > 80% tổng parasitic, làm crosstalk trở thành vấn đề số một của routing.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Formula: $\text{Pitch} = \text{Min Width} + \text{Min Spacing}$
→ Defined in: [[LEF]] — Tech LEF, LAYER section, PITCH keyword
→ Directly enables: [[Track]] · [[RoutingGrid]]
→ Links to Placement: Cell height = N × M1 Pitch → [[Site]] · [[Row]]
→ Affects: Routing density · RC parasitics · Coupling Capacitance
→ Cùng nhóm: [[Track]] · [[RoutingGrid]] · [[Site]] · [[Row]] · [[PlacementGrid]]