---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — internal classification của clock nets theo vị trí và fanout trong clock tree
used_by: [ClockTreeSynthesis, CTSOptimization, NDR, Routing]
requires: [ClockTreeSynthesis, NDR, RoutingGrid]
chain: Chain_PnR_Flow
---
# ClockTreeNetTypes

## Definition
Clock tree nets được phân loại thành ba loại dựa trên vị trí trong cây và transitive fanout đến các sinks. Phân loại này là cơ sở để CTS và Router áp dụng NDR rules, routing strategy, và layer assignment khác nhau cho từng phần của clock tree, cân bằng giữa signal integrity và routing resource consumption.

## Leaf Nets
Leaf net là bất kỳ net nào kết nối trực tiếp đến một hoặc nhiều clock tree sinks. Leaf nets nằm ở tầng cuối của clock tree — kết nối trực tiếp đến các stop pins (flip-flop clock pins, latch clock pins, macro clock pins).

Theo mặc định, CTS insert buffers sao cho không có buffer nào vừa drive sinks vừa drive internal nodes khác trên cùng net.

## Trunk Nets
Trunk net là tất cả nets không phải leaf nets (theo mặc định). Trunk nets tạo nên backbone của clock distribution network, kết nối từ clock source hoặc các buffer stages trung gian xuống đến pre-leaf level.

## Top Nets
Top net là tập con của trunk nets được xác định khi attribute `cts_routing_top_fanout_threshold` được configure. Bất kỳ trunk net nào có transitive fanout sink count vượt ngưỡng threshold đó sẽ được phân loại là top net thay vì trunk net.

Ví dụ: nếu threshold = 10,000, thì trunk net nào nằm phía trên 10,000 sinks hoặc nhiều hơn trong clock tree fan-in cone sẽ là top net.

## Relation to NDR Rules
Phân loại net type cho phép assign NDR rules theo mức độ tác động:
- **Root/Top nets**: ảnh hưởng nhiều sinks nhất → NDR nghiêm ngặt nhất (wider wire, larger spacing, shielding)
- **Internal/Trunk nets**: ảnh hưởng trung bình → NDR mức trung
- **Sink/Leaf nets**: ảnh hưởng một hoặc vài sinks → NDR nhẹ hơn hoặc default rules

Chiến lược này tối ưu trade-off: tập trung NDR overhead tại nơi có tác động lan rộng nhất, giảm routing resource waste ở các net cục bộ.

## Requires
- [[ClockTreeSynthesis]] — CTS phân loại nets trong quá trình build clock tree
- [[NDR]] — phân loại net type là cơ sở để assign NDR rules phù hợp
- [[RoutingGrid]] — clock nets được route trên RoutingGrid với constraints từ NDR

## Used by
- [[ClockTreeSynthesis]] — tool phân loại nets và assign routing strategy tương ứng
- [[CTSOptimization]] — optimization targeting có thể phân biệt theo net type
- [[NDR]] — NDR rules được apply theo net type classification
- [[Routing]] — clock net routing nhận routing rules dựa trên net type

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Closely related: [[NDR]] · [[ClockTreeSynthesis]] · [[CTSOptimization]] · [[ClockExceptions]]
→ Cùng nhóm: [[ClockTreeSynthesis]] · [[CTSFlow]] · [[NDR]] · [[ClockSkewGroup]]