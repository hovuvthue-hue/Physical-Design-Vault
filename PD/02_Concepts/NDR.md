---
tags: [concept, pnr-flow, routing, cts]
group: PnR Flow
defined_in: Routing — custom routing constraints cho selected nets, thường được áp dụng cho clock nets trong CTS context
used_by: [ClockTreeSynthesis, Routing, SignalIntegrity, Signoff]
requires: [Routing, RoutingGrid, Track, MetalStack]
chain: Chain_PnR_Flow
---
# NDR

## Definition
Non-Default Rule (NDR) là tập custom routing constraints override các default routing rules cho một số nets được chỉ định. Thay vì dùng minimum wire width và minimum spacing theo default tech rules, NDR cho phép chỉ định wider wire width, larger wire spacing, additional shielding, hoặc special via rules cho các nets đó. Khi một NDR được áp dụng cho một net, Router phải tuân thủ routing constraints của NDR đó cho net đó — không dùng default rules.

## Why Use NDR
NDR phục vụ ba mục đích chính:

**Improve Signal Integrity:**
Wider spacing giữa clock net và neighboring signal nets giảm coupling capacitance → giảm crosstalk noise trên clock. Clock nets đặc biệt nhạy cảm với crosstalk vì noise trên clock ảnh hưởng toàn bộ clock domain — xem [[SignalIntegrity]].

**Improve Reliability:**
Wider metal width giảm current density → giảm electromigration risk → tăng device lifetime — xem [[Electromigration]]. Quan trọng cho clock trunk và power-critical nets mang dòng lớn.

**Improve Timing:**
Lower resistance (wire rộng hơn → R thấp hơn → RC delay giảm) cải thiện timing trên critical nets. Shielding ổn định net delay bằng cách giảm coupling variation từ switching activity của neighboring nets.

## NDR for Clock Nets in CTS
Clock nets là đối tượng phổ biến nhất của NDR trong CTS context vì clock cần signal integrity cao nhất trong chip:
- NDR thường được áp dụng cho clock trunk và clock spine — các segments có fanout cao và ảnh hưởng rộng nhất
- Typical clock NDR pattern: 2× minimum width, 2× minimum spacing (thường ký hiệu 2W/2S)
- NDR rule được specify trong Clock Tree Spec và là một trong các inputs của [[ClockTreeSynthesis]]

## NDR Application by Clock Tree Net Type
Trong clock tree, NDR rules được áp dụng phân tầng theo loại net:

- **Root nets**: nets gần clock source, drive toàn bộ hoặc phần lớn clock tree → NDR nghiêm ngặt nhất, thường kết hợp wider wire + larger spacing + shielding
- **Internal nets**: nets trung gian trong clock tree → NDR mức trung bình
- **Sink nets**: nets kết nối trực tiếp đến clock sinks (leaf nets) → NDR nhẹ hơn hoặc default rules tùy design

Chiến lược này phù hợp với nguyên tắc: ảnh hưởng càng rộng thì protection requirement càng cao. Root net ảnh hưởng toàn bộ chip nếu bị noise — cần được bảo vệ chặt nhất. Sink net chỉ ảnh hưởng một hoặc vài flip-flops — overhead NDR đầy đủ có thể không cần thiết và lãng phí routing resources.

## NDR Trade-offs
Áp dụng NDR có chi phí:
- **Routing resources**: wire rộng hơn và spacing lớn hơn tiêu thụ nhiều routing tracks hơn → routing capacity giảm cho các nets khác → tăng congestion risk
- **Area**: NDR routes chiếm nhiều diện tích hơn default routes
- **Routing complexity**: Router phải tìm paths thỏa mãn NDR constraints trong môi trường tài nguyên bị hạn chế hơn

Vì vậy NDR thường chỉ áp dụng cho nets thực sự critical (clock trunk, critical timing nets), không phải toàn bộ design.

## Requires
- [[Routing]] — NDR là routing constraint được Router enforce trong detailed routing
- [[RoutingGrid]] — NDR quy định cách Router sử dụng routing grid (width và spacing trên grid)
- [[Track]] — wider NDR wires chiếm nhiều tracks hơn default wires
- [[MetalStack]] — NDR thường chỉ định layers cụ thể để áp dụng rule (ví dụ: M4/M5 cho clock nets)

## Used by
- [[ClockTreeSynthesis]] — NDR được specify trong clock tree spec để đảm bảo clock routes có signal integrity tốt
- [[Routing]] — Router enforce NDR constraints khi route các nets được assign NDR
- [[SignalIntegrity]] — NDR là phương tiện giảm crosstalk thông qua wider spacing
- [[Signoff]] — EM và SI checks xác nhận NDR đã đạt mục tiêu reliability và signal integrity

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Applies to: selected nets (clock nets, critical timing nets)
→ Defines: wire width · wire spacing · shielding requirements · via rules
→ Trade-off: signal integrity / reliability gain ↔ routing resource cost
→ Closely related: [[SignalIntegrity]] · [[ClockTreeSynthesis]] · [[Routing]] · [[Electromigration]]
→ Cùng nhóm: [[Routing]] · [[GlobalRouting]] · [[DetailedRouting]] · [[RoutingBlockage]]