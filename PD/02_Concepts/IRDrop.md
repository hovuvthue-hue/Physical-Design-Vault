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

1. **Static IR Drop**: thành phần sụt áp gắn với dòng tiêu thụ trung bình/ổn định theo thời gian tương đối dài.

**Static IR Drop formula:**

$$\text{Static IR Drop} = I_{Avg} \times R = \frac{P_{Avg}}{V_{DD}} \times R$$

Trong đó $P_{Avg}$ là Gate-level Average Power, gồm ba thành phần:

$$P_{Avg} = P_{Leakage} + P_{Internal} + P_{Switching}$$

| Thành phần | Công thức | Nguồn dữ liệu |
|---|---|---|
| $P_{Leakage}$ | Gate Leakage + Junction Leakage + Subthreshold Leakage | `.lib` (leakage_power) |
| $P_{Internal}$ | Internal Energy × Freq × Toggle Rate (TR) | `.lib` (internal_power) |
| $P_{Switching}$ | ½ × C × V² × Freq × TR | `.spef` (C), timing file (TR) |

Hệ quả: design tiêu thụ power cao hơn → $I_{Avg}$ lớn hơn → Static IR Drop lớn hơn → effective VDD tại cell thấp hơn → [[CellDelay]] tăng → rủi ro timing vi phạm tăng.


2. **Dynamic IR Drop**: thành phần sụt áp tức thời do switching activity đồng thời gây current spikes; thường đi kèm voltage droop/ground bounce trong các thời điểm nhạy cảm.

**Dynamic IR Drop** xảy ra khi nhiều cells chuyển trạng thái đồng thời tại high frequency, gây current spike vượt mức $I_{Avg}$. Hai nguồn gây ra:
- **Switching Noise**: switching current của các logic devices.
- **Rush Current Noise**: rush current khi charging up decoupling capacitors của local grid khi woken up.

Dynamic IR Drop tạo ra **Voltage Droop** trên VDD và **Ground Bounce** trên VSS — là voltage fluctuation tức thời ảnh hưởng cell delay trong khoảng thời gian ngắn sau clock edge.

## Ảnh hưởng tới timing
Khi IR Drop tăng, effective VDD tại cell giảm. Hệ quả điển hình:
- Cell có thể chuyển mạch chậm hơn (delay tăng),
- timing margin giảm,
- nguy cơ vi phạm timing tăng ở các path nhạy.

Vì vậy [[IRDrop]] có liên hệ trực tiếp đến [[CellDelay]], [[Slack]] và kết quả [[STA]].

## Phân tích timing failure theo vị trí IR drop

Vị trí xảy ra IR drop quyết định loại timing violation phát sinh — đây là điểm phân biệt quan trọng trong debug và fix:

**Trường hợp 1 — IR drop trên signal path:**
Cell trên data path nhận $V_{DD}$ thấp hơn thiết kế → khả năng drive current giảm → switching chậm lại → [[CellDelay]] tăng → [[Slack]] setup xấu đi → rủi ro **Setup violations** trên path đó.

**Trường hợp 2 — IR drop trên clock buffer:**
Clock buffer nhận $V_{DD}$ thấp hơn → clock signal phía sau buffer bị trễ thêm → tất cả FFs được clock bởi branch đó nhận clock edge muộn hơn so với launch side → [[ClockSkew]] tăng theo hướng bất lợi cho Hold → rủi ro **Hold violations** cho toàn bộ signals thuộc clock branch đó.

Cơ chế này giải thích tại sao IR drop phải được phân tích trên cả fabric điện và clock network — drop trong logic không tương đương drop trong clock tree về hậu quả timing.

## Quan hệ với PDN
[[PDN]] là nguyên nhân gốc về mặt kiến trúc phân phối nguồn:
- Lưới nguồn có đường cấp tốt hơn, kết nối đầy đủ hơn, điện trở hiệu dụng thấp hơn → rủi ro IR Drop giảm.
- Lưới nguồn kém liên tục hoặc nghẽn tài nguyên → rủi ro IR Drop tăng.

## Fix strategies cho Static IR Drop

Khi IR drop analysis (thường chạy qua `analyze_rail` tại Floorplan stage) phát hiện vấn đề:

- **Threshold thực tế**: vùng có voltage drop >10% nominal $V_{DD}$ thường được xem là cần fix ngay (ví dụ: $V_{DD}$ = 1.0V → drop >100 mV là vùng đỏ trong IR map). Ngưỡng chính xác phụ thuộc product/foundry requirement. [Needs verification]
- **Fix chính**: tăng width của power stripes, hoặc bổ sung metal trên các layer cao hơn (thick layers có sheet resistance thấp hơn → current carrying capacity cao hơn → $R$ giảm → $V_{drop}$ giảm).
- **Fix phụ**: thêm power stripes bổ sung tại vùng hotspot; trong IR drop analysis tool, các stripes được thêm thường được visualize bằng màu riêng (ví dụ cyan/magenta) để phân biệt với stripes gốc.

Nguyên tắc: tăng metal trên các layer được ưu tiên vì thick/ultra-thick layers vừa có resistance thấp hơn vừa ít ảnh hưởng routing resource signal.

## Quan hệ với Signoff và STA
- Ở mức flow, IR Drop được xem như một phần của power integrity trước khi tape-out trong [[Signoff]].
- Kết quả IR Drop ảnh hưởng gián tiếp tới timing closure vì delay của cell/net phụ thuộc điều kiện nguồn, do đó có liên quan tới [[STA]].

Không đặt ngưỡng pass/fail cụ thể trong card này vì budget phụ thuộc công nghệ, sản phẩm và methodology cụ thể. **[Needs verification]**

## Related
- Upstream: [[PDN]] · [[Floorplanning]] · [[Routing]]
- Downstream: [[Signoff]] · [[STA]]
- Timing links: [[CellDelay]] · [[Slack]]
