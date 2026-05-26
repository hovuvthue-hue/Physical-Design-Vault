---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — characterize bởi Foundry cho từng Flip-flop cell ở từng PVT corner
used_by: [STA, LogicSynthesis, ClockTreeSynthesis, Signoff]
requires: [Slack, SDC, CellDelay, NetDelay]
chain: Chain_STA_Basics
---
# SetupTime

## Definition
Setup Time (t_setup) là khoảng thời gian tối thiểu mà tín hiệu dữ liệu tại chân D của Flip-flop bắt buộc phải duy trì ổn định trước khi sườn xung nhịp kích hoạt (triggering clock edge) xuất hiện, để đảm bảo Flip-flop chốt đúng dữ liệu. Nếu dữ liệu thay đổi trong cửa sổ cấm này (Setup window), Flip-flop có thể rơi vào trạng thái Metastability — ngõ ra Q không xác định được là 0 hay 1. Setup Time là thành phần cuối cùng trong Setup Slack equation, đóng vai trò là "phí đầu cuối" mà data path phải trả tại Endpoint.

## Computed from
Setup Time được Foundry đo bằng SPICE simulation cho từng Flip-flop cell, lưu trong LIB file. Tương tự [[CellDelay]], t_setup là hàm của Input Data Transition (độ dốc tín hiệu D) và Clock Transition — không phải hằng số. LIB lưu t_setup dưới dạng 2D LUT: một trục là data transition time, trục kia là clock transition time. Setup check equation đầy đủ trong STA:

![[Pasted image 20260521090921.png]]

![[Pasted image 20260521091019.png|596]]

T_c + δ_skew > t_cq + t_logic_max + t_setup + δ_margin

Viết lại theo [[Slack]]:

Setup Slack = (T_c + δ_skew) − (t_cq + t_logic_max + t_setup + δ_margin) = RAT − AT

Trong đó t_setup làm giảm RAT — tức là thu hẹp thời gian dành cho data path. T_c (clock period) xuất hiện trong equation → Setup violation có thể được cứu sau silicon bằng cách giảm tần số (tăng T_c), dù chip sẽ chạy chậm hơn spec.

## Constrains
- **[[Slack]]**: t_setup là hằng số âm trong RAT computation — nó trực tiếp thu hẹp timing budget dành cho combinational logic; t_setup lớn hơn đồng nghĩa với ít room hơn cho Stage Delays trên data path
- **[[ClockTreeSynthesis]]**: δ_skew trong equation thể hiện positive skew có thể bù đắp cho t_setup (tăng timing budget) — đây là kỹ thuật "useful skew" trong CTS; tuy nhiên positive skew đồng thời làm trầm trọng thêm Hold violations
- **[[LogicSynthesis]]**: Synthesis tool optimize để Σ Stage_Delay ≤ T_c − t_cq − t_setup − δ_margin; t_setup là fixed overhead nên synthesis phải budget phần còn lại cho logic cloud

## Requires
- [[Slack]] — Setup Slack là metric tổng hợp bao gồm t_setup; để hiểu Setup Time cần hiểu cách nó đóng góp vào Slack equation
- [[SDC]] — `create_clock -period` định nghĩa T_c; `set_clock_uncertainty` định nghĩa δ_margin; cả hai cùng với t_setup xác định RAT
- [[CellDelay]] · [[NetDelay]] — cấu thành t_logic_max (tổng Stage Delays) — phần bên kia của inequality với t_setup; trade-off giữa logic speed và setup requirement

## Used by
- [[STA]] — annotate t_setup từ LIB lên timing graph tại mỗi Flip-flop Endpoint; dùng để compute RAT trong Backward Propagation; Setup check là default check type khi chạy `report_timing`
- [[LogicSynthesis]] — t_setup là constraint buộc synthesis phải minimize path delay; tool chọn faster cells (LVT, higher drive strength) trên paths với negative Setup Slack
- [[ClockTreeSynthesis]] — post-CTS STA recheck Setup Slack với real [[ClockSkew]]; một số Setup violations từ pre-CTS có thể được cải thiện nhờ positive skew của actual Clock Tree
- [[Signoff]] — Setup Signoff yêu cầu Setup Slack ≥ 0 tại tất cả slow-corner Analysis Views (SS process, low voltage, high/inverted temperature); là hard criterion cho Tape-out

## Key insight
[USER REVIEW — draft suggestion]: Setup Time tạo ra bất đối xứng quan trọng giữa hai loại timing violations: Setup violation là "bệnh của hiệu suất" — chip vẫn functional nếu giảm tần số, chỉ chậm hơn so với spec; ngược lại Hold violation là "bệnh chết người" — không có cách nào cứu sau silicon. Hiểu điều này giúp PD engineer phân loại ưu tiên: trong giai đoạn Routing, Setup violations được fix trước (timing closure), Hold violations được fix sau (buffer insertion) nhưng Hold fix phải hoàn toàn sạch trước Tape-out vì không có second chance. Thực tế quan trọng khác: t_setup trong LIB thường được measure tại ngưỡng 50% VDD — nếu Input Data Slew quá thoải (Max Transition violation), t_setup effective tăng lên và có thể tạo ra Setup violations bất ngờ không thấy trong typical conditions.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Equation: T_c + δ_skew > t_cq + t_logic_max + t_setup + δ_margin
→ Counterpart: [[HoldTime]] — cùng cặp timing checks của Flip-flop
→ Metric: [[Slack]] (Setup Slack = RAT − AT)
→ Stored in: [[LIB]]
→ Constrained by: [[SDC]] (T_c, δ_margin)
→ Fixed by: cell upsizing · logic restructuring · clock period relaxation · useful skew
→ Cùng nhóm: [[HoldTime]] · [[Slack]] · [[STA]] · [[SDC]] · [[CellDelay]] · [[NetDelay]]