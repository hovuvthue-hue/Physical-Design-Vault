---
tags: [concept, cell-library, pnr-flow, power]
group: Cell Library
defined_in: Standard Cell Library; inserted during Logic Synthesis hoặc Power Optimization
used_by: [PowerOptimization, PowerAnalysis, ClockTreeSynthesis, PathGrouping, STA, Signoff]
requires: [StandardCell, ClockTreeSynthesis, SDC]
chain: Chain_PnR_Flow
---
# ICG

## Definition
**ICG (Integrated Clock Gating Cell)** là Standard Cell chuyên dụng để gate clock signal một cách an toàn và không tạo glitch, bằng cách tích hợp một negative-level latch và một AND gate trong cùng một cell. ICG là cơ chế power optimization chính trong modern digital design: khi một register bank không cần capturing data, ICG gating signal low sẽ chặn clock đến toàn bộ group đó, loại bỏ hoàn toàn switching activity của clock path và giảm dynamic power một cách đáng kể.

## Traditional Clock Gating vs ICG

**Traditional Clock Gating** dùng AND gate trực tiếp: `Output = CLK AND Enable`. Vấn đề cốt lõi là glitch — nếu Enable thay đổi khi CLK đang high (gần clock edge), output tạo ra một xung ngắn không chủ ý. Glitch truyền xuống toàn bộ logic được clock bởi net đó, gây unnecessary switching và tăng power thay vì giảm.

**ICG Cell** giải quyết bằng latch gating:
- Latch là **negative-level transparent**: transparent khi CLK = 0, hold khi CLK = 1.
- Enable được capture và stabilized trong suốt thời gian CLK = 0.
- Khi CLK chuyển lên 1, latch đã hold giá trị Enable ổn định — AND gate output sạch, không glitch.

Nhờ cơ chế này, ICG cells là **preferred** trong modern designs cho robust, low-power clock gating, trong khi traditional AND gate gating không đảm bảo glitch-free operation.

## Why ICG is preferred

ICG mang lại bốn ưu điểm so với traditional AND gate gating: (1) **glitch-free** do Enable được stabilized trước clock edge; (2) **timing-safe** do ICG có characterization setup/hold riêng trên Enable pin trong LIB, đảm bảo Enable arrive đúng hạn; (3) **power reduction** đáng kể khi Enable = 0 vì toàn bộ downstream logic không toggle; (4) **EDA tool support** — STA, CTS, và power tools đều nhận biết ICG cells đặc biệt và xử lý chúng đúng cách.

## ICG in STA and PathGrouping

STA tool nhận diện ICG Enable pin qua attribute `is_clock_gating_check == true` trong LIB. Điều này tạo ra hai path groups riêng trong [[PathGrouping]]:
- **in2icg**: paths từ primary inputs đến ICG Enable pin — effort_level low.
- **reg2icg**: paths từ registers đến ICG Enable pin — effort_level high, vì ICG Enable paths ảnh hưởng trực tiếp đến clock domain power switching.

Setup/Hold check tại ICG Enable pin (clock gating check) được phân tích riêng biệt so với data register timing. Violation tại Enable pin có thể gây glitch (nếu Enable không stable đủ sớm trước clock edge) hoặc missed gating (nếu gating không được apply đúng cycle).

## ICG in ClockTreeSynthesis

CTS nhận diện ICG cells như **clock gating cells**, không phải regular logic. Gated clock output của ICG là một branch của clock tree và phải được route với chất lượng tương đương main clock về transition và skew. ICG cells thường được placed gần register banks chúng serve để minimize gated clock net length. Số lượng ICG cells ảnh hưởng trực tiếp đến tổng clock sink count và complexity của CTS.

## ICG in Power Optimization

ICG insertion là một trong những power optimization lever mạnh nhất để giảm dynamic power của clock network:

$$\Delta P_{dynamic} = \alpha_{saved} \cdot C_{FF+logic} \cdot V^2 \cdot f$$

Trong đó $\alpha_{saved}$ là switching activity được eliminate khi clock bị gated. Clock power của FF banks lớn (SRAM controllers, datapath registers) có thể chiếm phần đáng kể tổng clock power; ICG gating chặn hoàn toàn phần này khi logic không active.

ICG insertion thường xảy ra tại Logic Synthesis (RTL-level clock gating inference). Coverage và placement của ICG cells có thể được optimize thêm trong power optimization phase sau placement.

## Requires
- [[StandardCell]] — ICG là Standard Cell với characterization đặc biệt trong LIB bao gồm clock gating check arcs và latch timing.
- [[ClockTreeSynthesis]] — CTS phải nhận biết và xử lý ICG cells đặc biệt khi build và balance gated clock branches.
- [[SDC]] — Enable path timing constraints và clock gating checks được define trong SDC.

## Used by
- [[PowerOptimization]] — ICG là lever chính để giảm clock dynamic power.
- [[PowerAnalysis]] — clock gating coverage và effectiveness được report qua `report_clock_gating`.
- [[ClockTreeSynthesis]] — CTS phải build và balance gated clock nets từ ICG outputs.
- [[PathGrouping]] — in2icg và reg2icg là path groups riêng với effort_level high.
- [[STA]] — clock gating check tại ICG Enable pin là loại check riêng biệt, không phải standard setup/hold của data register.
- [[Signoff]] — clock gating coverage và timing correctness tại ICG Enable paths là signoff criteria.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Contrasted with: Traditional Clock Gating (AND gate only — glitch-prone).
→ Mechanism: negative-level latch + AND gate — Enable stabilized trong CLK = 0 phase.
→ Closely related: [[PowerOptimization]] · [[ClockTreeSynthesis]] · [[PathGrouping]] · [[STA]]
→ Cùng nhóm (Cell Library): [[StandardCell]] · [[TieCell]] · [[TapCell]] · [[SpareCell]]