---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Quantus (standalone) / Innovus (integrated)
used_by: [STA, Signoff]
requires: [Routing, LEF, GateLevelNetlist]
chain: Chain_PnR_Flow
---
# ParasiticExtraction

## Definition
Parasitic Extraction là bước trích xuất các giá trị điện trở (R) và điện dung (C) ký sinh từ wire và [[Via]] geometries thực tế trong Route DB. Các giá trị RC này không tồn tại trong Netlist logic — chúng phát sinh từ đặc tính vật lý của metal wires trên silicon. Output là file SPEF (Standard Parasitic Exchange Format) chứa [[InterconnectRC]] của từng Net, được STA tool tiêu thụ để tính wire delay chính xác.

## Computed from
Extraction tool tính toán parasitics dựa trên wire geometry và process parameters:

- **Resistance**: `R = ρ × L / (W × T)`
  - ρ = resistivity của metal layer (từ PDK)
  - L = wire length, W = wire width, T = wire thickness
  - Via resistance được extract riêng cho từng via stack
- **Capacitance**: phức tạp hơn R vì bao gồm:
  - **Area capacitance**: giữa wire và substrate bên dưới
  - **Fringe capacitance**: từ cạnh bên của wire
  - **Coupling capacitance**: giữa wire và neighboring wires trên cùng hoặc adjacent layers — đây là thành phần quan trọng nhất ở advanced nodes vì wire pitch nhỏ
- **Extraction modes:**
  - **RC (resistive + capacitive)**: mode chuẩn cho STA
  - **C-only**: nhanh hơn, dùng cho early estimation
  - **RCCT (coupled RC)**: extract coupling capacitance riêng biệt giữa aggressor/victim net pairs — cần thiết cho [[SignalIntegrity|Crosstalk]] analysis

## Constrains
- **STA accuracy**: chất lượng SPEF trực tiếp quyết định độ chính xác của post-route timing — SPEF sai hoặc thiếu sẽ tạo ra timing sign-off không hợp lệ
- **Signoff**: SPEF phải được extract ở nhiều Process Corners (best case, typical, worst case) để STA có thể run multi- corner analysis — thiếu bất kỳ corner nào là blocker cho Timing Signoff
- **[[SignalIntegrity|Crosstalk / SI]]**: coupling capacitance values trong SPEF là input cho Signal Integrity analysis — nếu coupling C lớn, [[SignalIntegrity|Crosstalk-induced delay]] và noise phải được analyzed riêng

## Requires
- [[Routing]] — wire geometries hoàn chỉnh và DRC-clean trong Route DB là input bắt buộc; extraction trên layout chưa DRC-clean cho kết quả không đáng tin cậy
- [[LEF]] — layer stackup definitions (metal thickness, dielectric constants, layer spacing) từ PDK LEF là parameters vật lý mà extraction tool dùng để tính R và C
- [[GateLevelNetlist]] — cung cấp net names và connectivity để tool map extracted RC values vào đúng Net trong SPEF; không có Netlist, SPEF không có context để STA tool đọc

## Used by
- [[STA]] (post-route, signoff) — đọc SPEF để annotate wire delays lên từng Net trong timing graph; đây là lần đầu tiên STA có đủ thông tin vật lý thực tế để tính [[Slack]] chính xác
- [[Signoff]] — SPEF là một trong các deliverables bắt buộc của Physical Design; Timing Signoff không thể thực hiện nếu thiếu SPEF hoặc SPEF chưa được verify

## Key insight
[USER REVIEW — draft suggestion]:
Parasitic Extraction là cầu nối giữa thế giới vật lý (wire geometry trong layout) và thế giới timing (delay trong STA). Trước bước này, mọi timing number đều là ước tính dựa trên wire length models; sau bước này mới có con số thực tế từ silicon. Điểm thực tế quan trọng: ở advanced nodes (dưới 16nm), coupling capacitance giữa các wires gần nhau chiếm tỷ trọng
ngày càng lớn trong tổng parasitic — đây là lý do Crosstalk analysis trở thành mandatory signoff check thay vì optional ở các node hiện đại.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Routing]]
→ Downstream: [[STA]] · [[Signoff]]
→ Output format: [[SPEF]]
→ Extracted concept: [[InterconnectRC]]
→ Closely related: [[SignalIntegrity|Crosstalk]] · [[SignalIntegrity]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[Signoff]]