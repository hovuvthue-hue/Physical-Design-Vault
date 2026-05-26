---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — bảng NLDM/CCS do Foundry characterize
used_by: [STA, LogicSynthesis, Signoff]
requires: [LIB, Slew, NetDelay]
chain: Chain_STA_Basics
---
# CellDelay

## Definition
Cell Delay (a.k.a. Gate Delay, Logic Delay) là thời gian tín hiệu truyền qua bên trong một Standard Cell, đo từ thời điểm tín hiệu ngõ vào đạt 50% VDD đến thời điểm tín hiệu ngõ ra đạt 50% VDD. Cell Delay không phải hằng số — nó là một hàm phi tuyến của 2 biến: Input Transition time (độ dốc tín hiệu đầu vào) và Output Load Capacitance (tổng điện dung tải ngõ ra). Mỗi Timing Arc trong một cell có Cell Delay riêng biệt và bất đối xứng giữa sườn lên (rise) và sườn xuống (fall).

![[Pasted image 20260521091457.png]]
## Computed from
Cell Delay được Foundry đo bằng SPICE simulation cho từng Standard Cell ở từng Process Corner, sau đó lưu vào LIB file dưới dạng 2D Look-up Table (LUT): một trục là Input Net Transition, trục kia là Output Load Capacitance. Khi STA tool cần giá trị tại một điểm không có trong bảng, nó dùng thuật toán nội suy 2 chiều (2D interpolation). Công thức đơn giản hóa: `Cell Delay = f(Input_Transition, C_load)` — trong đó C_load = C_net + C_pin của tất cả Receivers. Ở advanced nodes (< 130nm), mô hình CCS/ECSM thay thế NLDM: thay vì lookup một con số delay, tool tích phân dạng sóng dòng điện `i(t, V_out)` phi tuyến để reconstruct đường cong điện áp chính xác hơn.

![[Pasted image 20260521091513.png]]
## Constrains
- **[[NetDelay]]**: Input Transition của cell hiện tại trở thành output transition của cell, sau đó quyết định độ méo sóng trên Net — từ đó ảnh hưởng đến Cell Delay của stage tiếp theo (chuỗi Slew degradation)
- **[[StageDelay]]**: Cell Delay là thành phần thứ nhất trong Stage Delay; nếu Cell Delay quá lớn (cell yếu, load nặng), Stage Delay vượt budget → Setup violation
- **[[Signoff]]**: Cell Delay phải được tính ở tất cả Process Corners (SS/TT/FF) và nhiệt độ để MMMC signoff

## Requires
- [[LIB]] — chứa LUT 2D (NLDM) hoặc current waveform tables (CCS/ECSM) cho từng Timing Arc của cell; không có LIB, STA tool không thể tính Cell Delay
- [[Slew]] — Input Transition (Slew) của tín hiệu đầu vào là một trong hai trục của LUT; giá trị này được truyền xuống từ Output Slew của stage trước
- [[NetDelay]] — C_load trong công thức bao gồm C_net (điện dung dây dẫn sau cell); muốn tính đúng Cell Delay phải biết Net topology sau khi Routing

## Used by
- [[STA]] — đọc LIB để annotate Cell Delay lên từng Timing Arc trong timing graph; tổng hợp AT (Arrival Time) tại mỗi node
- [[LogicSynthesis]] — dùng Cell Delay từ LIB để chọn cell type phù hợp trong technology mapping và optimization; ở synthesis stage dùng wire-load model ước tính C_load
- [[Signoff]] — Timing Signoff tool (Tempus, PrimeTime) recompute Cell Delay với SPEF thực tế thay vì ước tính; đây là nguồn "correlation gap" giữa synthesis timing và post-route timing

## Key insight
[USER REVIEW — draft suggestion]: Cell Delay là nơi giao nhau giữa thế giới vật lý (transistor physics → LIB characterization) và thế giới timing (STA graph). Điểm thực tế quan trọng: cell càng to (drive strength X4, X8) thì C_load nhìn từ stage trước tăng (input capacitance lớn hơn), nhưng bản thân nó drive tải nhanh hơn — đây là trade-off cốt lõi khi engineer chọn cell size. Ở advanced nodes dưới 16nm, sự bất đối xứng rise/fall delay và non-linearity của CCS model càng rõ rệt, khiến việc estimate timing trước Routing ngày càng kém chính xác.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Closely related: [[NetDelay]] · [[StageDelay]] · [[Slew]] · [[TimingArc]]
→ Stored in: [[LIB]]
→ Used by: [[STA]] · [[LogicSynthesis]] · [[Signoff]]
→ Cùng nhóm: [[NetDelay]] · [[StageDelay]] · [[SetupTime]] · [[HoldTime]] · [[Slack]]