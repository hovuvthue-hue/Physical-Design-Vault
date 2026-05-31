---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Quantus (standalone) / Innovus (integrated)
used_by: [STA, Signoff, PostRouteOptimization]
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
  - **Coupling capacitance**: giữa wire và neighboring wires trên cùng hoặc adjacent layers — mức độ quan trọng tăng khi wire spacing nhỏ và phụ thuộc process/design style. [Needs verification]
- **Extraction modes:**
  - **RC (resistive + capacitive)**: mode chuẩn cho STA
  - **C-only**: nhanh hơn, dùng cho early estimation
  - **RCCT (coupled RC)**: extract coupling capacitance riêng biệt giữa aggressor/victim net pairs — có thể cần cho [[SignalIntegrity|Crosstalk]] analysis tùy methodology. [Needs verification]

## Constrains
- **STA accuracy**: chất lượng SPEF ảnh hưởng trực tiếp đến độ tin cậy của post-route timing; tiêu chí completeness/validity phụ thuộc extraction setup và signoff methodology. [Needs verification]
- **Signoff**: SPEF thường được extract theo các RC/process corners mà MMMC setup yêu cầu; coverage cụ thể và blocker criteria phụ thuộc project policy. [Needs verification]
- **[[SignalIntegrity|Crosstalk / SI]]**: coupling capacitance values trong SPEF là input cho Signal Integrity analysis — nếu coupling C lớn, [[SignalIntegrity|Crosstalk-induced delay]] và noise phải được analyzed riêng

## Requires
- [[Routing]] — wire geometries hoàn chỉnh và DRC-clean trong Route DB là input bắt buộc; extraction trên layout chưa DRC-clean cho kết quả không đáng tin cậy
- [[LEF]] — layer stackup definitions (metal thickness, dielectric constants, layer spacing) từ PDK LEF là parameters vật lý mà extraction tool dùng để tính R và C
- [[GateLevelNetlist]] — cung cấp net names và connectivity để tool map extracted RC values vào đúng Net trong SPEF; không có Netlist, SPEF không có context để STA tool đọc

## Used by
- [[STA]] (post-route, signoff) — đọc SPEF để annotate wire delays lên từng Net trong timing graph; đây là stage STA có parasitic data gắn với routed geometry để đánh giá [[Slack]] sát implementation hơn. [Needs verification]
- [[Signoff]] — SPEF là một deliverable quan trọng của Physical Design; Timing Signoff thường cần SPEF được kiểm tra theo methodology. [Needs verification]
- [[PostRouteOptimization]] — tiêu thụ extracted parasitics/SPEF sau khi routed geometry tồn tại để cleanup timing, DRV, và các chỉnh sửa post-route có kiểm soát

## Key insight
ParasiticExtraction là cầu nối giữa routed geometry trong layout và timing/power/SI analysis. Trước extraction, nhiều timing number vẫn dựa trên RC estimate; sau extraction, [[SPEF]] mang parasitic data từ routed wires/vias vào [[STA]] và các phân tích hậu route. Ở các process node tiên tiến hoặc các design có wire spacing nhỏ, coupling capacitance có thể trở thành thành phần quan trọng của parasitic model; mức độ quan trọng và yêu cầu [[SignalIntegrity|Crosstalk]]/SI analysis phụ thuộc process, design style và signoff methodology. [Needs verification]

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Routing]]
→ Downstream: [[STA]] · [[PostRouteOptimization]] · [[Signoff]]
→ Output format: [[SPEF]]
→ Re-analysis: route hoặc ECO-like change làm đổi geometry có thể cần extraction/STA lại tùy flow [Needs verification]
→ Extracted concept: [[InterconnectRC]]
→ Closely related: [[SignalIntegrity|Crosstalk]] · [[SignalIntegrity]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[Signoff]]
