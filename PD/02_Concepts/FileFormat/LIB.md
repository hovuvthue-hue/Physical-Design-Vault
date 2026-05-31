---
tags: [concept, file-format, sta-timing]
group: File Formats
defined_in: Foundry / Library vendor — Liberty format (.lib), industry standard do Synopsys định nghĩa
used_by: [STA, LogicSynthesis, ClockTreeSynthesis, Signoff]
requires: [N/A — LIB là input artifact từ Foundry characterization]
chain: Chain_STA_Basics
---
# LIB

## Definition
LIB (Liberty Timing Model, file extension .lib) là file text ASCII chứa timing, power, và noise models của tất cả Standard Cells và Hard IP Macros trong một technology library, được characterize ở một PVT corner cụ thể (Process, Voltage, Temperature). Mỗi file LIB đại diện cho đúng một corner — MMMC analysis cần nhiều LIB files (slow corner, fast corner, typical corner). LIB là bridge giữa thế giới vật lý (transistor characteristics đo bằng SPICE) và thế giới timing (delay numbers dùng trong STA) — nó cho phép STA tool tính Cell Delay trong milliseconds thay vì chạy SPICE simulation mất hàng giờ.

## Computed from
LIB được tạo qua quá trình Cell Characterization: Foundry chạy SPICE simulation cho từng Standard Cell trên một lưới các điểm (Input Slew × Output Load Capacitance), đo delay và transition time tại mỗi điểm, và lưu kết quả vào bảng 2D Look-up Table (LUT). Quá trình này được thực hiện cho từng PVT corner riêng biệt.

**Cấu trúc phân cấp của LIB file:**

Cấp 1 — `library(name)`: Operating conditions (Process/Voltage/Temperature), đơn vị đo lường, default thresholds (10%–90% cho Slew, 50% cho Delay).

Cấp 2 — `cell(cell_name)`: Area, leakage power, function description cho từng Standard Cell.

Cấp 3 — `pin(pin_name)`: Capacitance của từng pin, direction (input/output), clock pin flag.

Cấp 4 — `timing()`: Timing Arc data bao gồm:
- `cell_rise` / `cell_fall`: Propagation Delay LUT — $t_{pd} = f(\text{Input Slew}, C_{load})$
- `rise_transition` / `fall_transition`: Output Slew LUT
- `setup_rising` / `hold_rising`: Setup/Hold constraint LUT cho Flip-flop pins
- Timing sense (positive_unate, negative_unate, non_unate)

**NLDM vs CCS/ECSM**: NLDM lưu một số delay tại mỗi ô LUT; CCS (Synopsys) và ECSM (Cadence) lưu current waveforms hoặc voltage waveforms — chính xác hơn nhiều ở advanced nodes (< 130nm) nhưng file size lớn hơn hàng chục lần:

CCS (Synopsys) driver model:
- Captures output current flowing through load capacitor
- Yêu cầu non-zero capacitance tại output của cell
- STA tool nhìn input voltage waveform, dùng CCS model để tính output current,
  sau đó dùng output load để compute voltage waveform → extract delay

ECSM (Cadence) driver model:
- Captures voltage waveform tại output của cell
- Có thể bắt đầu với zero load capacitance

Receiver Model (cả hai):
- Captures Miller capacitance với sensitivity to slope và load

## Constrains
- **[[CellDelay]]**: LIB chứa LUT data mà STA tool dùng để compute Cell Delay; không có LIB, không thể tính Cell Delay
- **[[Slew]]**: LIB chứa Output Slew LUT song song với Cell Delay LUT; STA tool propagate Slew qua timing graph bằng cách lookup LIB
- **[[SetupTime]]** / **[[HoldTime]]**: Setup Arc và Hold Arc trong LIB define t_setup và t_hold của từng Flip-flop; là thành phần của RAT computation trong STA
- **[[STA]]**: LIB là primary timing data source; quality của LIB characterization (NLDM vs CCS) quyết định accuracy của STA results; MMMC cần LIB files cho tất cả corners
- **[[ClockTreeSynthesis]]**: CTS tool dùng LIB để chọn Clock Buffer cells với balanced rise/fall delay

## Requires
N/A — LIB là primary input artifact được Foundry characterize từ SPICE models. LIB cho một corner cụ thể (ví dụ SS 0.9V 125°C) hoàn toàn phụ thuộc vào process node characteristics của Foundry, không phụ thuộc vào design.

## Used by
- [[STA]] — đọc LIB để annotate Cell Delay, Output Slew, Setup/Hold constraints lên timing graph; mỗi Timing Arc được annotate bằng LUT values từ LIB; MMMC STA cần LIB cho tất cả corners
- [[LogicSynthesis]] — đọc LIB trong technology mapping step để chọn Standard Cell phù hợp với timing/area/power objectives; synthesis iterate qua LIB lookups hàng triệu lần trong optimization loops
- [[ClockTreeSynthesis]] — đọc LIB của Clock Buffer/Inverter cells để verify Slew và Insertion Delay trong budget; chọn cell với balanced rise/fall delay
- [[Signoff]] — Signoff STA tool (Tempus, PrimeTime) đọc LIB ở tất cả MMMC corners; kết quả signoff chỉ valid nếu LIB đã được properly characterized

## Key insight
[USER REVIEW — draft suggestion]: LIB là nơi physics meets math — SPICE simulation (physics) được "đóng gói" thành lookup tables (math) để STA tool có thể chạy analysis trong vài phút thay vì vài tuần. Điểm quan trọng cần hiểu về MMMC: một design cần nhiều LIB files vì delay của cùng một cell thay đổi đáng kể theo PVT — một Inverter ở FF corner (fast process, high voltage, low temperature) có thể nhanh gấp 3 lần so với SS corner (slow process, low voltage, high temperature). Setup check cần SS corner LIB (cell chậm nhất); Hold check cần FF corner LIB (cell nhanh nhất). Dùng sai LIB cho sai corner là một trong những lỗi nghiêm trọng nhất trong STA setup vì nó không gây error message — STA vẫn chạy và report kết quả, chỉ là kết quả không đúng.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Format: Liberty (.lib) — industry standard do Synopsys định nghĩa
→ Timing models: NLDM (older, lightweight) · CCS/ECSM (advanced nodes, accurate)
→ Contains: [[CellDelay]] LUT · [[Slew]] LUT · [[SetupTime]] arc · [[HoldTime]] arc · [[TimingArc]] definitions
→ One file per: PVT corner (SS/TT/FF × voltage × temperature)
→ Used with: [[LEF]] (physical) · [[GDS]] (layout) — bộ 3 cần thiết cho IP integration
→ Consumed by: [[STA]] · [[LogicSynthesis]] · [[ClockTreeSynthesis]] · [[Signoff]]
→ Cùng nhóm: [[LEF]] · [[SPEF]] · [[DEF]] · [[GDS]]