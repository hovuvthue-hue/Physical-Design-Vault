---
tags: [concept, technology-file, routing, interconnect]
group: Technology File
defined_in: LEF / Technology File
used_by: [Routing, ParasiticExtraction, Signoff]
requires: [MetalStack, LEF, ITF]
chain: Chain_LEF_to_PnR
---
# Via

## Definition
Via là cấu trúc interconnect theo chiều dọc dùng để nối hai metal layers liền kề trong [[MetalStack]]. Nếu wire trên một net cần đổi từ layer này sang layer khác, Router phải đặt Via tại vị trí hợp lệ để tạo kết nối điện liên tục giữa hai đoạn wire.

Via không phải là một net logic mới; nó là phần hình học và điện học của routed interconnect. Vì vậy Via vừa bị ràng buộc bởi routing rules trong [[LEF]], vừa đóng góp parasitic vào [[InterconnectRC]] sau khi [[ParasiticExtraction]] chạy.

## Why Via exists in Routing
[[Routing]] dùng nhiều metal layers để tránh congestion, đi vòng quanh blockage, truy cập pin, hoặc đưa net lên layer phù hợp hơn về timing và resource. Mỗi lần net chuyển layer, Router cần một Via để nối đoạn wire trên layer dưới với đoạn wire trên layer trên.

Trong [[RoutingGrid]], wire đi theo [[Track]] trên từng layer, còn Via là kết nối dọc giữa các layer tại vị trí hợp lệ. [[Pitch]] và track spacing ảnh hưởng đến việc có đủ không gian hợp lệ để đặt Via, đặc biệt ở vùng congestion cao.

## Relation to MetalStack and routing layers
[[MetalStack]] định nghĩa thứ tự các metal layers và môi trường vật lý giữa chúng. Via là phần tử nối cặp layer liền kề trong stack đó. [[LEF]] mô tả các ràng buộc hình học/routing rule cho layer và via; [[ITF]] cung cấp dữ liệu công nghệ để extraction hiểu tác động điện học của metal/via geometry.

Các quy tắc như enclosure, cut spacing, via type, hoặc cách chọn redundant/multi-cut via phụ thuộc vào PDK, foundry, tool setting, process node, và signoff policy cụ thể [Needs verification]. Card này chỉ mô tả vai trò khái niệm của Via, không áp đặt rule hay giá trị số.

## Electrical impact
Via thêm resistance và capacitance vào interconnect. Khi một routed net dùng nhiều Via, phần đóng góp này có thể làm [[NetDelay]] tăng, ảnh hưởng [[Slew]], và làm [[Slack]] thay đổi sau post-route extraction.

[[ParasiticExtraction]] dùng routed wire/via geometry cùng dữ liệu công nghệ để tạo [[SPEF]]. Trong SPEF, tác động của Via được phản ánh như một phần của extracted parasitics, thay vì là ước lượng trừu tượng trước Routing.

## Reliability / manufacturability considerations
Via là điểm nhạy cảm về manufacturability và reliability vì nó phải kết nối chính xác giữa hai metal layers. Via count, vị trí Via, và lựa chọn single-cut hay redundant/multi-cut via có thể ảnh hưởng tới delay, congestion, yield, và long-term reliability.

Redundant hoặc multi-cut via thường được dùng ở mức khái niệm để tăng độ tin cậy hoặc giảm rủi ro tại điểm chuyển layer, nhưng chính sách áp dụng cụ thể là tool/PDK/foundry/flow-specific [Needs verification]. Việc thêm nhiều Via cũng tiêu thụ routing resource và có thể làm congestion hoặc DRC closure khó hơn.

## Requires
- [[MetalStack]] — xác định các metal layers mà Via kết nối theo chiều dọc.
- [[LEF]] — chứa routing/via rules ở mức công nghệ để Router biết vị trí và hình học hợp lệ.
- [[ITF]] — cung cấp dữ liệu công nghệ để extraction tính tác động điện học của interconnect.

## Used by
- [[Routing]] — cần Via mỗi khi net đổi layer hoặc truy cập pin qua layer khác.
- [[ParasiticExtraction]] — đọc routed wire/via geometry để tính parasitics thực tế.
- [[SPEF]] — lưu kết quả parasitic đã extract từ routed interconnect, bao gồm đóng góp của Via.
- [[Signoff]] — kiểm tra timing, manufacturability, và reliability dựa trên layout và parasitics sau Routing.

## Related
→ Stack context: [[MetalStack]]
→ Geometry/rules: [[LEF]] · [[RoutingGrid]] · [[Track]] · [[Pitch]]
→ Electrical model: [[InterconnectRC]] · [[ITF]]
→ Extracted into: [[ParasiticExtraction]] → [[SPEF]]
