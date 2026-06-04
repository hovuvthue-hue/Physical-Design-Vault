---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: Manually written TCL file; dùng trong cả Logic Synthesis và PnR flow
used_by: [MMMC, STA, LogicSynthesis, DesignImport, Placement, ClockTreeSynthesis, Routing, Signoff]
requires: [GateLevelNetlist]
chain: Chain_STA_Basics
---
# SDC

## Definition
Synopsys Design Constraints (SDC) là format file định nghĩa toàn bộ timing requirements và exceptions của một design. Một số tài liệu đào tạo đôi khi gọi là 'Synopsys Delay Constraints' (không chính xác) hoặc 'Standard Delay Constraints' (tên không chính thức nhưng phổ biến). Format file: **TCL**. SDC được viết thủ công, thường bởi Synthesis Engineer, và được dùng lại (với điều chỉnh nhỏ) qua cả Synthesis và PnR flow.

SDC hoạt động trên **Design Objects** — các thực thể mà timing constraints tham chiếu đến. Phân biệt chính xác các Design Objects là bắt buộc để SDC hoạt động đúng:

| Object    | Định nghĩa                                                                        | Scope                       |
| --------- | --------------------------------------------------------------------------------- | --------------------------- |
| Design    | Mô tả mạch thực hiện một chức năng (Verilog module)                               | module boundary             |
| Cell      | Instantiation của Design trong Design khác (Verilog instance)                     | bên trong Design            |
| Reference | Design gốc mà Cell "trỏ đến" (Verilog sub-module)                                 | logical definition          |
| Port      | input/output/inout của **Design** — tức là boundary I/O của module                | module boundary             |
| Pin       | input/output/inout của **Cell** bên trong Design — tức là instance-level terminal | bên trong Design            |
| Net       | Wire kết nối Ports với Pins và/hoặc Pins với nhau                                 | physical connectivity       |
| Clock     | Port hoặc Pin được tường minh định nghĩa là clock source trong SDC                | xác định bởi `create_clock` |

**Phân biệt Port vs Pin**: Port là I/O ở module-level boundary (block interface); Pin là I/O ở instance-level terminal (cell connection). Cùng một signal có thể là Port của module cha và là driver của Net kết nối đến Pins của các cell instances bên trong. Sự nhầm lẫn Port/Pin là nguyên nhân phổ biến của "object not found" errors trong SDC.

**Phân biệt Clock source trong SDC**: Clock phải được tường minh define — nếu PnR tool không tìm thấy clock definition tại một endpoint, endpoint đó trở thành "unconstrained" và không được check trong STA.

SDC chứa các loại constraints:

**Clock constraints**: định nghĩa clock period, waveform, uncertainty (Jitter + Skew budget). Clock period trực tiếp xác định RAT cho tất cả Setup checks; ClockUncertainty là safety margin chống lại Jitter và estimated Skew.

**I/O constraints**: định nghĩa arrival time của data tại input Ports và required time của data tại output Ports. Thiếu I/O constraints tạo ra unconstrained I/O paths — STA không check những paths này.

**Timing exceptions**: false path (path không cần check timing), multicycle path (path được phép dùng nhiều clock cycles), clock groups (clocks không synchronous với nhau). Exceptions sai hoặc missing exceptions đều gây silicon failure.

Multicycle path không phải là “fix” cho một path chậm theo nghĩa tự làm path đó nhanh hơn; nó chỉ hợp lệ khi design intent thật sự cho phép data được capture sau nhiều chu kỳ. Nếu một path không meet one-cycle timing nhưng architecture/protocol cố ý cấp nhiều cycle cho path đó, SDC cần model intent này bằng multicycle path constraint. Nếu path về mặt chức năng vẫn là one-cycle, áp multicycle là sai và có thể che giấu timing violation thật. Useful skew là lever liên quan ở CTS/timing-closure, nhưng không phải timing exception trong SDC.

**Physical constraints**: `set_max_transition` đặt upper bound cho Slew; `set_max_capacitance` đặt upper bound cho Net load.

## Computed from

Timing Sanity — lý thuyết rút ra từ SDC structure:

Setup check formula với SDC parameters:
$$\text{Setup Slack} = (T_c + \delta_{skew}) - (t_{cq} + t_{logic,max} + t_{setup} + \delta_{uncertainty,setup})$$

Hold check formula với SDC parameters:
$$\text{Hold Slack} = (t_{ccq} + t_{logic,min} - \delta_{margin}) - (t_{hold} + \delta_{skew})$$

Trong đó $T_c$ đến từ `create_clock -period`; $\delta_{uncertainty}$ đến từ `set_clock_uncertainty`; $\delta_{skew}$ là physical measurement (pre-CTS: estimated từ SDC, post-CTS: propagated từ actual tree).

**Timing path types** (xác định bởi SDC I/O constraints):

| Path type | Start point | End point | Constrained bởi |
|---|---|---|---|
| Register-to-Register | Flip-flop clock Pin | Flip-flop data Pin | `create_clock` |
| Input-to-Register | Input Port | Flip-flop data Pin | `set_input_delay` |
| Register-to-Output | Flip-flop clock Pin | Output Port | `set_output_delay` |
| Input-to-Output | Input Port | Output Port | `set_input_delay` + `set_output_delay` |

## Constrains
- **[[SetupTime]]**: clock period và setup uncertainty trong SDC xác định RAT cho Setup check; tighter period → tighter RAT → harder Setup
- **[[HoldTime]]**: hold uncertainty trong SDC xác định required minimum data path delay; sau CTS, `set_propagated_clock` thay thế ideal clock model để expose real Skew
- **[[Slew]]**: `set_max_transition` đặt upper bound cho Slew trên toàn design; violation = buffer insertion required
- **[[MMMC]]**: mỗi Constraint Mode trong MMMC file chứa một SDC file; Operating Mode (Functional, Scan, Test) tương ứng một SDC
- **[[ClockUncertainty]]**: uncertainty value trong SDC là combined budget của Jitter + worst expected Skew trước CTS

## Requires
- [[GateLevelNetlist]] — SDC references specific Port names, cell instance names, và clock Pin names; SDC phải được viết với knowledge về design hierarchy để object names resolve correctly

## Used by
- [[MMMC]] — mỗi Constraint Mode embed một SDC file
- [[STA]] — primary consumer: dùng SDC để compute RAT, identify exceptions, set clock period
- [[LogicSynthesis]] — đọc SDC để guide Technology Mapping và Optimization
- [[Placement]] → [[ClockTreeSynthesis]] → [[Routing]] — mỗi stage đọc SDC để guide timing-driven optimization; SDC được refine giữa các stages
- [[Signoff]] — MMMC Signoff cần SDC cho từng Constraint Mode; coverage check verify SDC completeness

## Key insight
SDC là nơi design intent được chuyển thành tập constraint mà EDA tools dùng để phân tích và tối ưu timing. Nếu SDC thiếu constraint, một số path có thể trở thành unconstrained và không được STA đánh giá đúng; nếu timing exceptions sai, STA có thể bỏ qua các path đáng lẽ phải được check. Vì vậy, coverage/analysis-coverage review là bước quan trọng trước khi tin vào STA result. Sự khác biệt giữa Synthesis SDC và PnR SDC cũng có thể tạo ra timing-closure mismatch giữa các stage hoặc giữa các team. Mức độ rủi ro và cách kiểm tra phụ thuộc tool/flow cụ thể. [Needs verification]

## Related
→ Chain: [[Chain_STA_Basics]]
→ Design Objects: Design · Cell · Reference · Port · Pin · Net · Clock
→ Path types: Reg-to-Reg · Input-to-Reg · Reg-to-Output · Input-to-Output
→ Consumed by: [[MMMC]] · [[STA]] · [[LogicSynthesis]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[Signoff]]
→ Defines constraints for: [[SetupTime]] · [[HoldTime]] · [[ClockSkew]] · [[ClockUncertainty]] · [[Slew]]
→ Related formats: [[SPEF]] (physical parasitics) · [[LIB]] (cell timing) — cùng là inputs của STA
→ Cùng nhóm: [[STA]] · [[CellDelay]] · [[NetDelay]] · [[Slack]] · [[MMMC]]

## Related clock concepts
- [[VirtualClock]]: mốc clock logic cho timing I/O khi không có clock source vật lý trong design.
- [[GeneratedClock]]: clock dẫn xuất từ master clock, cần mô hình đúng quan hệ clock cho phân tích timing.
