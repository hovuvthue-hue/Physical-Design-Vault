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
Pitch là khoảng cách từ tâm đến tâm giữa hai Track (tương ứng vị trí center-line của wire hợp lệ) liền kề trên cùng một metal layer. Pitch là thông số nền tảng nhất của Routing Grid — nó xác định mật độ tối đa của các wires có thể được đặt trên một metal layer mà không vi phạm DRC spacing rules. Pitch được Foundry định nghĩa cho từng metal layer trong Tech LEF và là hằng số cố định cho một process node — PD engineer không thể thay đổi giá trị này.

## Computed from
$$\text{Pitch} = \text{Min Wire Width} + \text{Min Wire Spacing}$$

Trong đó:
- **Min Wire Width**: chiều rộng tối thiểu của dây dẫn trên layer đó (đủ để dòng điện chạy qua mà không bị electromigration, đủ để lithography in được)
- **Min Wire Spacing**: khoảng cách tối thiểu giữa hai cạnh ngoài của hai wires liền kề (đủ để tránh rò điện và đảm bảo process yield)

Ví dụ từ L6: trong TRACKS statement, giá trị `STEP` chính là Pitch của layer tương ứng.

Pitch thay đổi theo layer và theo process/node; card này chỉ giữ ở mức khái niệm, không áp đặt giá trị số cố định cho mọi công nghệ.

## Constrains
- **[[Track]]**: mỗi Track là một đường thẳng trên Routing Grid; khoảng cách giữa các Tracks liên tiếp trên cùng một layer = Pitch; số lượng Tracks trên một đoạn chiều dài L = L / Pitch
- **[[RoutingGrid]]**: toàn bộ Routing Grid được xây dựng bằng cách đặt các Tracks cách nhau đúng một Pitch; Pitch nhỏ hơn thường cho nhiều Tracks hơn trên cùng diện tích, nhưng cũng làm tradeoff congestion/coupling nhạy hơn ở các vùng demand cao
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
→ Affects: Routing capacity/congestion tradeoff · RC parasitics · Coupling behavior
→ Cùng nhóm: [[Track]] · [[RoutingGrid]] · [[Site]] · [[Row]] · [[PlacementGrid]]