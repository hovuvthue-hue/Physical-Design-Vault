---
tags: [concept, manufacturing, routing, signoff]
group: Manufacturing
defined_in: Routing / Signoff
used_by: [DetailedRouting, DRC, Signoff]
requires: [Routing, DetailedRouting, LEF, DEF]
chain: Chain_PnR_Flow
---
# AntennaEffect

## Definition
AntennaEffect là rủi ro manufacturing trong đó interconnect đang floating trong quá trình fabrication có thể tích điện và gây stress hoặc damage lên gate oxide đã nối với net đó. Đây là vấn đề liên quan cách layout được hình thành qua các bước manufacturing, không chỉ là connectivity cuối cùng sau khi chip hoàn tất.

Antenna rule ratios, model charge accumulation và pass/fail criteria là foundry/PDK/signoff-policy-specific. [Needs verification]

## Why antenna appears during routing/manufacturing
Trong [[Routing]], wire và [[Via]] geometry quyết định chiều dài/diện tích metal tạm thời gắn với gate trong các bước fabrication. Một đoạn interconnect dài hoặc một route order bất lợi có thể làm net dễ tích charge trước khi có đường xả phù hợp. [Needs verification]

Vì [[DetailedRouting]] tạo geometry cụ thể, routing decisions như layer jump, via placement hoặc route detour có thể ảnh hưởng antenna risk.

## Relation to DetailedRouting and DRC
[[DetailedRouting]] có thể cần antenna-aware repair nếu flow bật hoặc yêu cầu xử lý antenna trong routing stage. Tuy nhiên final antenna checking/cleanup có thể thuộc [[Signoff]] hoặc physical verification tùy methodology. [Needs verification]

[[DRC]] liên quan vì antenna thường được quản lý trong cùng physical-verification ecosystem, nhưng [[AntennaEffect]] là concept riêng: nó tập trung vào rủi ro charge/gate-oxide trong manufacturing, không chỉ spacing/width/enclosure geometry.

## Common repair concepts
Các repair concept thường gặp gồm:

- jumper insertion lên metal layer cao hơn để thay đổi exposure path;
- diode insertion để tạo đường discharge;
- routing changes như reroute hoặc tách đoạn metal nhạy;
- antenna rule cleanup trong routing/signoff loop.

Chọn repair nào, insert ở đâu, rule ratio nào cần thỏa và cell nào được phép dùng đều là foundry/PDK/tool/signoff-policy-specific. [Needs verification]

## Limits / tool dependence
- Card này không định nghĩa antenna ratio, diode rule hoặc repair command.
- Không thay thế antenna deck, signoff rule manual hoặc foundry guidance.
- Antenna-clean trong route stage không bảo đảm final signoff antenna clean nếu layout tiếp tục thay đổi hoặc rule environment khác. [Needs verification]

## Requires
- [[Routing]] — tạo physical interconnect geometry.
- [[DetailedRouting]] — cụ thể hóa wire/via paths có thể ảnh hưởng antenna risk.
- [[LEF]] — cung cấp abstract/library/technology context liên quan pins, layers và antenna data. [Needs verification]
- [[DEF]] — lưu placed/routed design geometry để downstream checks sử dụng.

## Used by
- [[DetailedRouting]] — có thể cần antenna-aware repair trong route closure. [Needs verification]
- [[DRC]] — physical verification context có thể bao gồm antenna-related checks. [Needs verification]
- [[Signoff]] — final flow có thể kiểm tra antenna như một manufacturing/reliability gate. [Needs verification]

## Related
→ Flow: [[Routing]] · [[DetailedRouting]] · [[Signoff]]
→ Physical verification: [[DRC]]
→ File context: [[LEF]] · [[DEF]]
