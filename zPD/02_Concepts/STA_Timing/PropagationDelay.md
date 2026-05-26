---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — characterize bởi Foundry cho từng Timing Arc của mỗi cell
used_by: [STA, CellDelay, SetupTime, HoldTime, StageDelay]
requires: [LIB, TimingArc]
chain: Chain_STA_Basics
---
# PropagationDelay

## Definition
Propagation Delay ($t_{pd}$) và Contamination Delay ($t_{cd}$) là hai giới hạn thời gian trái chiều của cùng một Timing Arc trong một cell — chúng xác định respectively thời điểm muộn nhất và sớm nhất mà output của cell thay đổi sau khi input thay đổi. $t_{pd}$ (a.k.a. Max Delay) là thời gian lâu nhất tín hiệu cần để truyền qua cell — đo từ 50% VDD của input đến 50% VDD của output khi output đạt trạng thái ổn định cuối cùng. $t_{cd}$ (a.k.a. Min Delay, Contamination Delay) là thời gian ngắn nhất trước khi output bắt đầu thay đổi — output có thể chưa ổn định nhưng đã bắt đầu rời khỏi trạng thái cũ. Sự phân biệt giữa $t_{pd}$ và $t_{cd}$ là nền tảng của Setup vs Hold analysis trong STA.

## Computed from
Cả hai đều được Foundry characterize bằng SPICE simulation và lưu trong LIB dưới dạng 2D LUT (Input Slew × Output Load Capacitance). Rise và Fall được lưu riêng biệt cho cả hai loại delay.

$$t_{pd} = \text{thời gian từ 50\% input đến 50\% output (worst-case — output đã stable)}$$

$$t_{cd} = \text{thời gian từ 50\% input đến lúc output bắt đầu thay đổi (best-case)}$$

Trong thực tế: $t_{cd} \leq t_{pd}$ luôn luôn đúng với mọi cell và mọi conditions. Khoảng chênh lệch $(t_{pd} - t_{cd})$ phụ thuộc vào cấu trúc logic bên trong cell — cell đơn giản (Inverter, Buffer) có khoảng chênh lệch nhỏ; cell phức tạp (Full Adder, MUX) có khoảng chênh lệch lớn hơn vì tín hiệu phải đi qua nhiều transistor stages.

Với Flip-flop, hai đại lượng tương ứng là:
- **Clock-to-Q Propagation Delay** ($t_{pcq}$): thời gian muộn nhất từ clock edge đến Q output ổn định — dùng cho Setup check
- **Clock-to-Q Contamination Delay** ($t_{ccq}$): thời gian sớm nhất Q bắt đầu thay đổi sau clock edge — dùng cho Hold check

## Constrains
- **[[SetupTime]]**: $t_{pd}$ (Max Delay) được dùng trong Setup check — STA tính $t_{logic,max} = \sum t_{pd,i}$ trên data path; Setup violation xảy ra khi $t_{pcq} + t_{logic,max} + t_{setup} > T_c + \delta_{skew}$
- **[[HoldTime]]**: $t_{cd}$ (Min Delay) được dùng trong Hold check — STA tính $t_{logic,min} = \sum t_{cd,i}$ trên data path; Hold violation xảy ra khi $t_{ccq} + t_{logic,min} < t_{hold} + \delta_{skew}$
- **[[STA]]**: hai luồng phân tích song song trong STA tương ứng trực tiếp với cặp này — MAX path analysis dùng $t_{pd}$, MIN path analysis dùng $t_{cd}$

## Requires
- [[LIB]] — cả $t_{pd}$ và $t_{cd}$ được lưu trong LIB dưới dạng separate LUT tables; STA tool đọc đúng table tùy theo đang chạy MAX hay MIN analysis
- [[TimingArc]] — $t_{pd}$ và $t_{cd}$ là properties của một Timing Arc cụ thể — không tồn tại độc lập mà luôn gắn với một arc (ví dụ: arc từ pin A đến pin Y của NAND2 với rising input)

## Used by
- **[[STA]]**: MAX analysis traverse timing graph dùng $t_{pd}$ tại mỗi Cell Timing Arc để compute AT_max → Setup Slack. MIN analysis traverse dùng $t_{cd}$ để compute AT_min → Hold Slack. Hai analyses chạy song song trong cùng một STA run
- **[[CellDelay]]**: CellDelay trong context Setup check = $t_{pd}$ của arc được traverse; CellDelay trong context Hold check = $t_{cd}$ — cùng một concept "Cell Delay" nhưng lấy giá trị khác nhau tùy loại check
- **[[StageDelay]]**: Stage Delay cho Setup = $t_{pd,cell} + t_{net}$; Stage Delay cho Hold = $t_{cd,cell} + t_{net,min}$

## Key insight
[USER REVIEW — draft suggestion]: Sự tồn tại song song của $t_{pd}$ và $t_{cd}$ phản ánh một sự thật vật lý quan trọng: không có một giá trị "delay của cell" duy nhất — delay phụ thuộc vào góc nhìn. Kỹ sư cần luôn hỏi: "delay này dùng để check gì — Setup hay Hold?" Điểm thực tế: trong timing report, khi thấy một path bị Hold violation dù các Stage Delays trông không quá nhỏ, nguyên nhân có thể là $t_{cd}$ thực tế nhỏ hơn nhiều so với $t_{pd}$ — timing report mặc định chỉ hiển thị $t_{pd}$; phải chạy `-early` report để thấy $t_{cd}$ values thực sự được dùng trong Hold analysis.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Stored in: [[LIB]] — separate LUT tables cho $t_{pd}$ và $t_{cd}$
→ Property of: [[TimingArc]]
→ $t_{pd}$ used in: [[SetupTime]] analysis (MAX path)
→ $t_{cd}$ used in: [[HoldTime]] analysis (MIN path)
→ Aggregated into: [[StageDelay]] · [[CellDelay]]
→ Cùng nhóm: [[CellDelay]] · [[TimingArc]] · [[Slew]] · [[SetupTime]] · [[HoldTime]]