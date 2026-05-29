---
tags:
  - concept
  - front-end
  - interface
group: Front-End Design
defined_in: "Cadence Genus / Synopsys Design Compiler (output: Verilog Gate-Level Netlist)"
Cadence Genus / Synopsys Design Compiler (output: Verilog Gate-Level Netlist)
used_by:
  - GateLevelNetlist
  - Floorplanning
  - Placement
  - STA
requires:
  - RTL
  - SDC
  - LIB
chain: Chain_BackEnd_Overview
---
# LogicSynthesis

## Definition
Logic Synthesis là quá trình tự động chuyển đổi RTL description (Verilog/VHDL) thành Gate-Level Netlist bằng cách ánh xạ logic design vào các Standard Cells từ một technology library cụ thể, đồng thời tối ưu hóa design để đáp ứng Timing, Area, và Power constraints được định nghĩa trong SDC. Logic Synthesis là bước cuối của Front-End Design và là điểm khởi đầu của Back-End Design — output của nó (Gate-Level Netlist) là input chính của toàn bộ PnR flow.

![[Pasted image 20260521090408.png]]
## Computed from
Logic Synthesis diễn ra theo 3 giai đoạn tuần tự trong synthesis tool:

- **Translation**: RTL code được parse và chuyển thành generic Boolean logic (AND/OR/NOT network) không phụ thuộc technology — giai đoạn này phát hiện các RTL coding issues như latch inference không chủ ý, undriven signals, multiple drivers
- **Technology Mapping**: generic Boolean logic được ánh xạ vào Standard Cells thực tế từ LIB file — tool chọn cell type (drive strength, logic function) dựa trên timing requirements của từng path; đây là giai đoạn quyết định chất lượng Netlist
- **Optimization**: tool iterate để cải thiện PPA — gate sizing (chọn drive strength phù hợp), logic restructuring (thay đổi topology để shorten critical path), buffer insertion (fix high-fanout nets) — tất cả được guide bởi SDC constraints

## Constrains
- **GateLevelNetlist quality**: chất lượng optimization trong Logic Synthesis quyết định điểm xuất phát của PnR — Netlist có critical path depth lớn, nhiều high-fanout nets, hoặc cell count quá lớn sẽ tạo ra khó khăn cho Placement và Timing closure trong PnR
- **Timing correlation**: Logic Synthesis dùng wire-load models (statistical estimates) thay vì actual parasitics — timing results từ synthesis luôn khác với post-route STA; khoảng cách này cần được account bằng cách set aggressive timing margins (guardband) trong SDC
- **Area estimation**: cell area trong Netlist là lower bound cho Core area trong Floorplanning — synthesis có thể được chạy lại với relaxed constraints để trade timing for area nếu Floorplanning bị area-constrained

## Requires
- [[RTL]] — Register-Transfer Level description là input bắt buộc; RTL quality (coding style, pipeline depth, module hierarchy) ảnh hưởng trực tiếp đến synthesis results và runtime
- [[SDC]] — Timing constraints (clock definitions, input/output delays, multicycle paths, false paths) là primary guide cho optimization; SDC không đầy đủ hoặc sai sẽ tạo ra Netlist không đáp ứng được timing requirements trong PnR
- [[LIB]] — Liberty Timing Model cung cấp timing ([[CellDelay]], transition time), area, và power data cho từng Standard Cell — synthesis tool không thể thực hiện technology mapping nếu thiếu LIB

## Used by
- [[GateLevelNetlist]] — output trực tiếp của Logic Synthesis; mọi downstream PnR steps đều consume Netlist này
- [[Floorplanning]] — đọc Netlist để ước tính area và phân tích block connectivity
- [[STA]] — synthesis tool chạy internal STA sau mỗi optimization iteration để verify timing constraints được đáp ứng trước khi output Netlist; đây là pre-layout STA với estimated parasitics

## Key insight
[USER REVIEW — draft suggestion]:
Logic Synthesis là bước Front-End duy nhất có tác động trực tiếp và sâu rộng đến toàn bộ PnR flow — một RTL design được synthesize tốt (timing margins hợp lý, no high-fanout nets, hierarchy phù hợp với Floorplan) tạo ra PnR flow smooth; ngược lại, synthesis kém chất lượng tạo ra cascade of problems từ Floorplanning đến Signoff không thể resolve hoàn toàn ở Back-End. Điểm quan trọng với người mới: Logic Synthesis và Physical Design thường do hai team khác nhau thực hiện — interface giữa hai team chính là Gate-Level Netlist và SDC; hiểu rõ hai artifacts này là hiểu rõ ranh giới trách nhiệm trong một tổ chức IC design thực tế.

## Related
→ Chain: [[Chain_BackEnd_Overview]]
→ Produces: [[GateLevelNetlist]]
→ Inputs: [[RTL]] · [[SDC]] · [[LIB]]
→ Closely related: [[StandardCell]] · [[TimingClosure]] · [[WireLoadModel]]
→ Cùng nhóm (Front-End): [[RTL]] · [[ArchitecturalDesign]] · [[MicroarchitectureDesign]]
→ Downstream chain: [[Chain_PnR_Flow]]