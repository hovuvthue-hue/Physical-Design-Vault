---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — characterize bởi Foundry; constrained bởi SDC (set_max_transition)
used_by: [STA, CellDelay, NetDelay, ClockTreeSynthesis, LogicSynthesis, Signoff]
requires: [LIB, SDC]
chain: Chain_STA_Basics
---
# Slew

## Definition
Slew (a.k.a. Transition Time) là thời gian tín hiệu cần để chuyển trạng thái từ mức logic thấp lên mức logic cao (Rise Time) hoặc ngược lại (Fall Time). Trong môi trường VLSI, không có tín hiệu kỹ thuật số nào chuyển trạng thái tức thời — tín hiệu phải nạp/xả các tụ điện ký sinh trên mạch nên luôn có độ dốc hữu hạn. Slew được đo trong khoảng từ 10% đến 90% VDD (hoặc 20%–80% tùy convention của Foundry). Slew là biến số đầu vào của [[CellDelay]] LUT và là output được compute bởi STA tool tại mỗi node — nó lan truyền qua toàn bộ timing path theo cơ chế Slew degradation.

## Computed from
Slew tại ngõ ra của một cell được STA tool tính từ LIB — tương tự Cell Delay, Output Slew được lưu dưới dạng 2D LUT với hai trục là Input Slew và Output Load Capacitance:

$$t_{slew} = f(\text{Input Slew},\ C_{load})$$

Rise Slew và Fall Slew được lưu riêng biệt vì PMOS network (kéo lên) và NMOS network (kéo xuống) có đặc tính dòng điện khác nhau — đây là nguồn gốc của tính bất đối xứng (asymmetry) trong timing.

**Slew Degradation** là hiện tượng quan trọng nhất liên quan đến Slew: Output Slew của cell này trở thành Input Slew của cell tiếp theo. Khi tín hiệu đi qua nhiều tầng logic hoặc net dài, độ dốc sườn tín hiệu giảm dần (Slew tăng lên — sườn thoải hơn). Slew càng lớn:
- Cell Delay của stage sau tăng (tra LUT với Input Slew lớn hơn → delay lớn hơn)
- Short-circuit power tăng (transistor ở trạng thái linear lâu hơn)
- Nguy cơ vi phạm Max Transition DRV

Cụ thể trên Net: dây dài với RC lớn làm méo sóng → Output Slew tại receiver tệ hơn Output Slew tại driver. STA tool model điều này bằng Slew degradation tables trong LIB.

## Constrains
- **[[CellDelay]]**: Input Slew là một trong hai trục của Cell Delay LUT — Slew lớn (thoải) → Cell Delay lớn; đây là cơ chế mà Slew degradation truyền ảnh hưởng qua chuỗi stages
- **[[NetDelay]]**: RC trên Net làm thoải sóng tín hiệu → tăng Slew tại receiver pin; Net dài là nguyên nhân chính của Slew degradation trong physical design
- **[[ClockTreeSynthesis]]**: Clock Slew là quality metric của CTS — clock signal phải có Slew đủ dốc (nhỏ) tại tất cả Flip-flop clock pins để đảm bảo clean triggering; Max Clock Transition trên clock nets/sinks là một trong các constraints cốt lõi của CTS
- **[[Signoff]]**: Max Transition DRV violation (Slew vượt quá giới hạn cho phép) là blocker cho Signoff — tín hiệu quá thoải có thể gây short-circuit power không kiểm soát được và functional failure

## Requires
- [[LIB]] — Output Slew LUT được lưu trong LIB song song với Cell Delay LUT; STA tool tra LIB để compute Output Slew tại mỗi cell output pin; không có LIB không thể propagate Slew qua timing graph
- [[SDC]] — `set_max_transition value [current_design]` thiết lập giới hạn Slew tối đa cho tất cả signals; `set_clock_transition value [get_clocks clk]` thiết lập giới hạn riêng cho clock network; đây là DRV constraints

## Used by
- [[STA]] — propagate Slew từ Startpoints đến Endpoints song song với AT computation; Output Slew tại mỗi node được dùng làm Input Slew cho LUT lookup của node tiếp theo; Max Transition violations được report cùng với timing violations
- [[CellDelay]] — Input Slew là biến đầu vào của Cell Delay computation; Slew degradation là feedback mechanism giữa NetDelay và CellDelay trong chuỗi stages
- [[ClockTreeSynthesis]] — CTS phải đảm bảo Clock Transition (Clock Slew) tại mọi FF clock pin nằm trong budget; Clock Buffers được chọn và sized để maintain Slew dọc theo Clock Tree
- [[LogicSynthesis]] — synthesis tool check Max Transition DRV và insert buffers để fix violations; buffer insertion để fix Slew là một trong các optimization steps trong synthesis
- [[Signoff]] — Max Transition DRV check là một trong các Signoff criteria; violations phải được fixed trước Tape-out

## Key insight
[USER REVIEW — draft suggestion]: Slew là khái niệm kết nối thế giới vật lý (transistor physics, wire RC) với thế giới timing (Cell Delay LUT lookup). Người mới thường chỉ quan tâm đến delay numbers mà bỏ qua Slew — nhưng trong thực tế, một timing path fail đôi khi không phải vì logic quá chậm mà vì Slew degradation tích lũy qua nhiều stages làm tăng Cell Delay ở cuối path. Dấu hiệu nhận biết: trong timing report, nếu cột Slew/Transition tăng dần và đột ngột vọt lên ở một stage nào đó → stage đó có driver quá yếu hoặc net quá dài → cần upsize cell hoặc insert repeater. Max Transition violation ảnh hưởng trực tiếp timing, đồng thời tăng rủi ro về power/noise/reliability [Needs verification] nên cần được xử lý sớm.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Stored in: [[LIB]] (Output Slew LUT)
→ Constrained by: [[SDC]] — set_max_transition · set_clock_transition
→ Feeds into: [[CellDelay]] (Input Slew → LUT axis)
→ Degraded by: [[NetDelay]] (RC trên wire làm thoải sóng)
→ Critical for: [[ClockTreeSynthesis]] (Clock Transition budget)
→ Cùng nhóm: [[CellDelay]] · [[NetDelay]] · [[StageDelay]] · [[STA]] · [[SDC]]