---
tags: [concept, pnr-flow, power-integrity]
group: PnR Flow
defined_in: Power analysis
used_by: [Floorplanning, Signoff, STA]
requires: [PDN, Routing, ParasiticExtraction]
chain: Chain_PnR_Flow
---
# IRDrop

## Definition
IR Drop là hiện tượng sụt áp trên đường phân phối nguồn do điện trở hữu hạn của lưới nguồn. Ở mức cơ bản:

$$V_{drop} = I \times R$$

Trong đó, khi dòng tiêu thụ tăng hoặc đường cấp nguồn có điện trở hiệu dụng cao, mức sụt áp tại cell sẽ tăng.

## Static IR Drop vs Dynamic IR Drop
- **Static IR Drop**: thành phần sụt áp gắn với dòng tiêu thụ trung bình/ổn định theo thời gian tương đối dài.
- **Dynamic IR Drop**: thành phần sụt áp tức thời do switching activity đồng thời gây current spikes; thường đi kèm voltage droop/ground bounce trong các thời điểm nhạy cảm.

## Ảnh hưởng tới timing
Khi IR Drop tăng, effective VDD tại cell giảm. Hệ quả điển hình:
- Cell có thể chuyển mạch chậm hơn (delay tăng),
- timing margin giảm,
- nguy cơ vi phạm timing tăng ở các path nhạy.

Vì vậy [[IRDrop]] có liên hệ trực tiếp đến [[CellDelay]], [[Slack]] và kết quả [[STA]].

## Quan hệ với PDN
[[PDN]] là nguyên nhân gốc về mặt kiến trúc phân phối nguồn:
- Lưới nguồn có đường cấp tốt hơn, kết nối đầy đủ hơn, điện trở hiệu dụng thấp hơn → rủi ro IR Drop giảm.
- Lưới nguồn kém liên tục hoặc nghẽn tài nguyên → rủi ro IR Drop tăng.

## Quan hệ với Signoff và STA
- Ở mức flow, IR Drop được xem như một phần của power integrity trước khi tape-out trong [[Signoff]].
- Kết quả IR Drop ảnh hưởng gián tiếp tới timing closure vì delay của cell/net phụ thuộc điều kiện nguồn, do đó có liên quan tới [[STA]].

Không đặt ngưỡng pass/fail cụ thể trong card này vì budget phụ thuộc công nghệ, sản phẩm và methodology cụ thể. **[Needs verification]**

## Related
- Upstream: [[PDN]] · [[Floorplanning]] · [[Routing]]
- Downstream: [[Signoff]] · [[STA]]
- Timing links: [[CellDelay]] · [[Slack]]
