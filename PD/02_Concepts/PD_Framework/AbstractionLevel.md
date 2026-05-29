---
tags: [concept, pd-framework]
group: PD Framework
defined_in: N/A — framework khái niệm theo Gajski Y-Chart, không thuộc file format hay tool cụ thể
used_by: [LogicSynthesis, Floorplanning, Placement, Routing]
requires: [N/A]
chain: Chain_BackEnd_Overview
---
# AbstractionLevel

## Definition
Abstraction Level là cấp độ biểu diễn của một IC design, trong đó mỗi cấp độ cao hơn ẩn đi chi tiết của cấp độ thấp hơn và thay bằng các simplifications. Trong VLSI design flow, có 5 cấp độ từ cao đến thấp: **Behavioral** (mô tả chức năng bằng thuật toán), **RTL** (mô tả bằng registers và data transfers), **Gate Level** (mô tả bằng Standard Cell instances và Net connections), **Circuit/Transistor Level** (mô tả bằng transistors và analog characteristics), **Physical Level** (mô tả bằng geometric shapes trên silicon layers). Physical Design hoạt động tại Physical Level nhưng nhận input từ Gate Level.

![[Pasted image 20260522223306.png|816]]

![[Pasted image 20260522223352.png]]
## Computed from
Abstraction Levels được formalize bởi **Gajski Y-Chart** — một mô hình 3 chiều giao nhau: Behavioral Domain (mô tả chức năng), Structural Domain (mô tả kết nối), và Physical Domain (mô tả geometry). Tại mỗi cấp độ abstraction, design được biểu diễn từ cả 3 góc nhìn này đồng thời. Chuyển đổi giữa các cấp là các tool-driven transformations: RTL → Gate Level bởi Logic Synthesis tool; Gate Level → Physical Level bởi PnR tool.

## Constrains
- **LogicSynthesis**: hoạt động tại ranh giới RTL → Gate Level — RTL (Behavioral/Structural) được map sang Standard Cells (Structural/Physical)
- **Floorplanning và Placement**: hoạt động tại Physical Level — Gate Level Netlist (connectivity) được ánh xạ sang physical coordinates (x, y) trên silicon
- **Routing**: hoàn thành quá trình chuyển đổi sang Physical Level — Net connections trong Netlist trở thành wire geometries thực tế
- **Verification**: mỗi transition giữa Abstraction Levels phải được verify để đảm bảo equivalence — LVS verify Gate Level ↔ Physical Level; Formal Verification verify RTL ↔ Gate Level

## Requires
N/A — Abstraction Level là framework lý thuyết mô tả cách IC design được biểu diễn và xử lý qua các bước khác nhau. Không phụ thuộc vào concept kỹ thuật upstream.

## Used by
- [[LogicSynthesis]] — thực hiện transformation từ RTL Abstraction Level xuống Gate Level Abstraction Level
- [[Floorplanning]] · [[Placement]] · [[Routing]] — ba bước này cùng thực hiện transformation từ Gate Level xuống Physical Level, mỗi bước add thêm một lớp physical information
- [[GateLevelNetlist]] — artifact tồn tại tại Gate Level Abstraction, là ranh giới interface giữa Front-End và Back-End

## Key insight
[USER REVIEW — draft suggestion]:
Hiểu Abstraction Levels giúp trả lời câu hỏi "tool này đang làm gì với design?" — Logic Synthesis tool không biết gì về wire lengths hay silicon geometry; PnR tool không đọc RTL; Signoff STA tool chỉ thấy timing graph trích xuất từ Netlist và SPEF. Mỗi tool chỉ hoạt động tại đúng abstraction level của nó và chỉ cần thông tin tại level đó. Đây là lý do tại sao interface giữa các tools (GateLevelNetlist, SPEF, DEF, GDS) quan trọng hơn bản thân các tools — chúng là điểm chuyển tiếp giữa các Abstraction Levels.

## Related
→ Chain: [[Chain_BackEnd_Overview]]
→ Transitions được thực hiện bởi: [[LogicSynthesis]] (RTL → Gate) · [[Floorplanning]] · [[Placement]] · [[Routing]] (Gate → Physical)
→ Artifacts tại mỗi level: [[RTL]] (RTL Level) · [[GateLevelNetlist]] (Gate Level) · [[DEF]] · [[GDS]] (Physical Level)
→ Framework reference: Gajski Y-Chart (CMOS VLSI Design — Weste-Harris)
→ Cùng nhóm: [[PPARMP]]