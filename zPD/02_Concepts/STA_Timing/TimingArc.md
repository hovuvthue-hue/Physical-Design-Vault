---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: LIB (Liberty file) — định nghĩa cho từng cell; modeled trong timing graph của STA tool
used_by: [STA, CellDelay, Slew, LogicSynthesis]
requires: [LIB, GateLevelNetlist]
chain: Chain_STA_Basics
---
# TimingArc

## Definition
Timing Arc là đơn vị cơ bản nhất của timing graph trong STA — nó đại diện cho mối quan hệ nhân quả (causal relationship) giữa một input pin và một output pin của một cell, hoặc giữa driver pin và receiver pin của một net. Nếu sự thay đổi tín hiệu tại một input pin trực tiếp dẫn đến sự thay đổi tín hiệu tại output pin, đường truyền ảo giữa hai pins đó là một Timing Arc. Một timing path từ Launch FF đến Capture FF là chuỗi các Timing Arcs nối tiếp nhau — Cell Timing Arcs xen kẽ với Net Timing Arcs.

## Computed from
Có hai loại Timing Arc trong timing graph:

**Cell Timing Arc**: Nằm bên trong cell — từ input pin đến output pin. Mỗi cặp (input pin, output pin) có thể có nhiều Timing Arcs tương ứng với các transition directions khác nhau:
- Rising input → Rising output (Positive Unate, ví dụ Buffer)
- Rising input → Falling output (Negative Unate, ví dụ Inverter)
- Falling input → Rising output (Negative Unate)
- Falling input → Falling output (Positive Unate)

Cell Timing Arc được lưu trong LIB với hai thông số per arc: [[PropagationDelay|Propagation Delay]] ($t_{pd}$) và Output Slew — cả hai đều là 2D LUT của (Input Slew × Output Load).

**Unateness** xác định chiều lan truyền signal:
- **Positive Unate**: input tăng → output tăng (AND, OR, Buffer)
- **Negative Unate**: input tăng → output giảm (NAND, NOR, Inverter)
- **Non-Unate**: output không thể xác định direction chỉ từ một input (XOR)

**Net Timing Arc**: Nằm trên net — từ driver output pin đến receiver input pin. Delay = Net Delay từ RC parasitics (SPEF). Net Timing Arcs luôn Positive Unate (signal truyền qua wire không đảo chiều) và không có intrinsic delay trong LIB — delay đến từ SPEF.

**Timing Arc trên Flip-flop** có cấu trúc đặc biệt:
- **Clock-to-Q Arc**: từ clock pin đến Q output — delay = $t_{cq}$ (Startpoint của data path)
- **Setup Arc**: từ data pin D đến clock pin — không phải [[PropagationDelay|Propagation Delay]] mà là constraint (minimum time D phải stable trước clock edge)
- **Hold Arc**: tương tự Setup Arc nhưng là minimum time D phải stable sau clock edge

## Constrains
- **[[CellDelay]]**: mỗi Cell Timing Arc có delay riêng — Cell Delay của một stage là delay của Timing Arc được kích hoạt trên path đó; một cell có thể có nhiều arcs nhưng chỉ một arc được traverse trên mỗi specific path
- **[[Slew]]**: Output Slew được compute per Timing Arc — arc từ input A đến output Y của NAND2 có Output Slew LUT riêng biệt với arc từ input B đến output Y; asymmetry giữa rise và fall arcs xuất phát từ đây
- **[[STA]]**: timing graph được build bằng cách connect các Timing Arcs thành chuỗi; Forward Propagation traverse các arcs theo chiều thuận; một số arcs có thể bị disable bởi `set_false_path` hoặc `set_case_analysis` trong SDC

## Requires
- [[LIB]] — Cell Timing Arcs và tất cả delay/slew LUTs được định nghĩa trong LIB; LIB là nguồn dữ liệu duy nhất cho Cell Timing Arc properties; Net Timing Arcs lấy delay từ SPEF
- [[GateLevelNetlist]] — Netlist xác định connectivity — cells nào kết nối với cells nào; từ connectivity này STA tool biết cần build Net Timing Arcs nào giữa các Cell Timing Arcs

## Used by
- [[STA]] — build timing graph từ tập hợp tất cả Timing Arcs trong design; traverse graph để compute AT và RAT; disable arcs theo SDC exceptions; report violated paths là các chuỗi arcs có negative Slack
- [[CellDelay]] — Cell Delay là giá trị delay của một Cell Timing Arc cụ thể được kích hoạt trên timing path; không có khái niệm Cell Delay mà không có Timing Arc tương ứng
- [[Slew]] — Output Slew được compute per arc và propagated sang Input Slew của arc tiếp theo; arc direction (rise/fall) quyết định STA dùng rise hay fall LUT entry
- [[LogicSynthesis]] — synthesis tool model timing arcs từ LIB để guide optimization; khi restructure logic, synthesis thay đổi set of arcs trên critical paths

## Key insight
[USER REVIEW — draft suggestion]: Timing Arc là abstraction quan trọng giúp STA scale lên hàng tỷ gates — thay vì simulate actual waveforms, STA chỉ cần traverse một graph các arcs và làm phép cộng. Điểm thực tế quan trọng: một cell có thể có nhiều input pins nhưng trên một specific timing path, chỉ có một arc được traverse (arc từ input pin nằm trên critical path đến output pin). Khi kỹ sư thấy một timing report với nhiều stages qua cùng một cell theo các paths khác nhau, mỗi path thực ra traverse một arc khác nhau với delay khác nhau. Hiểu Unateness giúp kỹ sư predict signal polarity trên long paths — quan trọng khi debug functional issues kết hợp với timing issues.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Defined in: [[LIB]] (Cell Timing Arcs) · [[SPEF]] (Net Timing Arcs delay values)
→ Building block of: [[CellDelay]] · [[Slew]] · [[STA]] timing graph
→ Types: Cell Arc (intra-cell) · Net Arc (inter-cell) · Setup Arc · Hold Arc · Clock-to-Q Arc
→ Controlled by: [[SDC]] — set_false_path · set_case_analysis (disable specific arcs)
→ Cùng nhóm: [[CellDelay]] · [[Slew]] · [[NetDelay]] · [[StageDelay]] · [[STA]]