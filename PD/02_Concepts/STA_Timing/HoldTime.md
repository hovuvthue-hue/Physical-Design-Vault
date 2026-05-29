---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — characterize bởi Foundry cho từng Flip-flop cell ở từng PVT corner
used_by: [STA, ClockTreeSynthesis, Routing, Signoff]
requires: [Slack, SDC, CellDelay, NetDelay]
chain: Chain_STA_Basics
---
# HoldTime

## Definition
Hold Time (t_hold) là khoảng thời gian tối thiểu mà tín hiệu dữ liệu tại chân D của Flip-flop bắt buộc phải tiếp tục duy trì ổn định sau khi sườn xung nhịp kích hoạt đã diễn ra, để đảm bảo Flip-flop hoàn tất việc chốt dữ liệu cũ trước khi nhận dữ liệu mới. Nếu data path quá nhanh (dữ liệu mới đến trong cửa sổ cấm t_hold), Flip-flop đang chốt dữ liệu cũ sẽ bị nhiễu bởi dữ liệu mới — dẫn đến Metastability hoặc ghi sai dữ liệu. Đặc điểm phân biệt cốt lõi: Hold Time check xảy ra trên cùng một sườn xung nhịp (same clock edge), hoàn toàn độc lập với clock period T_c.

## Computed from
Hold Time được Foundry đo bằng SPICE simulation, lưu trong LIB file dưới dạng 2D LUT (data transition × clock transition). Hold check equation đầy đủ trong STA:

![[Pasted image 20260521091057.png]]

![[Pasted image 20260521091114.png|590]]

t_ccq + t_logic_min − δ_margin > t_hold + δ_skew

Viết lại theo [[Slack]]:

Hold Slack = AT_min − RAT_hold = (t_ccq + t_logic_min − δ_margin) − (t_hold + δ_skew)

Trong đó:
- t_ccq: [[PropagationDelay|Contamination Delay]] của Launch FF (thời gian ngắn nhất dữ liệu mới thoát khỏi Q)
- t_logic_min: tổng Min Delay qua combinational logic (dùng [[PropagationDelay|Contamination Delay]], không phải [[PropagationDelay|Propagation Delay]])
- δ_margin: OCV derating làm cho fast paths trông nhanh hơn thực tế (bi quan về Hold)
- δ_skew: [[ClockSkew]] — positive skew làm Hold Slack xấu đi

T_c hoàn toàn vắng mặt trong equation → giảm tần số thường không trực tiếp giải quyết bản chất Hold violations. Hold violation trên silicon là rủi ro chức năng nghiêm trọng và cần đánh giá trong context thiết kế cụ thể. [Needs verification]

## Constrains
- **[[Slack]]**: Hold Slack âm là rủi ro chức năng nghiêm trọng và thường phải được clean trước Tape-out; khả năng workaround sau silicon phụ thuộc design/flow và không nên được giả định. [Needs verification]
- **[[ClockTreeSynthesis]]**: [[ClockSkew]] (δ_skew) xuất hiện với dấu cộng trong Hold check — positive skew có thể làm Hold Slack xấu đi. Sau CTS, khi Clock Tree thực tế được build và Skew không còn là zero, hold risk thường lộ rõ hơn; cách xử lý và mức độ ảnh hưởng phụ thuộc flow/project constraints. [Needs verification]
- **[[Routing]]**: Hold fix buffers được insert trên data paths ngắn để kéo dài AT_min; Routing phải accommodate vị trí của các hold buffers này; post-route Hold STA với fast-corner SPEF là Hold Signoff checkpoint cuối cùng

## Requires
- [[Slack]] — Hold Slack = AT_min − RAT_hold; t_hold là thành phần của RAT_hold; hiểu Hold Time cần đặt trong context của Slack equation
- [[SDC]] — `set_clock_uncertainty -hold` định nghĩa δ_margin riêng cho Hold check (thường nhỏ hơn Setup uncertainty vì Jitter = 0 cho Hold — cùng clock edge, không có inter-cycle jitter); `set_clock_groups` loại bỏ Hold check giữa async clock domains
- [[CellDelay]] · [[NetDelay]] — t_logic_min dùng [[PropagationDelay|Contamination Delay]] (Min Delay) thay vì [[PropagationDelay|Propagation Delay]] (Max Delay); short paths với ít logic gates và short wires có t_logic_min nhỏ → dễ Hold violation

## Used by
- [[STA]] — Hold check chạy song song với Setup check; tool dùng MIN paths (best-case delays) thay vì MAX paths; sau CTS, hold risk thường được quan sát rõ hơn do propagated clock và skew thực tế.
- [[ClockTreeSynthesis]] — post-CTS là checkpoint quan trọng để đánh giá và xử lý hold risk khi [[ClockSkew]] thực tế đã được phản ánh; cách và thời điểm fix phụ thuộc flow/project constraints. [Needs verification]
- [[Routing]] — Hold fix buffers cần routing resources; timing-driven routing phải đảm bảo Hold buffers được route với đủ wire length để giữ nguyên delay characteristics; post-route Hold STA với fast-corner SPEF là final verification
- [[Signoff]] — Hold Signoff thường yêu cầu Hold Slack không âm ở các fast-corner Analysis Views; chi tiết tiêu chí acceptance phụ thuộc signoff policy của project/tool flow. [Needs verification]

## Key insight
HoldTime thường bị đánh giá thấp vì không phụ thuộc clock period như Setup, nhưng vi phạm Hold là rủi ro chức năng nghiêm trọng và thông thường cần được clean trước Tape-out. Khả năng workaround sau silicon không nên được giả định trước, vì phụ thuộc mạnh vào kiến trúc thiết kế, cơ chế điều khiển clock, và flow debug thực tế. [Needs verification]

Một điểm thực tế quan trọng là post-CTS thường làm hold risk lộ rõ hơn do [[ClockSkew]] và propagated clock được mô hình hóa đầy đủ hơn trong [[STA]]. Việc hold fixing (ví dụ tăng data-path delay) cần cân bằng chặt với [[Slack]] setup để tránh tạo violation mới.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Equation: t_ccq + t_logic_min − δ_margin > t_hold + δ_skew
→ Counterpart: [[SetupTime]] — cùng cặp timing checks; Setup phụ thuộc T_c, Hold không
→ Metric: [[Slack]] (Hold Slack = AT_min − RAT_hold)
→ Stored in: [[LIB]]
→ Worst after: [[ClockTreeSynthesis]] — [[ClockSkew]] introduced
→ Fixed by: delay buffer insertion trên data paths · [[ClockSkew]] optimization · [[CTSOptimization]] context
→ Cùng nhóm: [[SetupTime]] · [[Slack]] · [[STA]] · [[SDC]] · [[CellDelay]] · [[NetDelay]]
