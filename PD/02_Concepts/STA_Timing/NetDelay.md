---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: SPEF (post-route) / Wire Load Model (pre-route) — extracted từ routing geometry
used_by: [STA, ParasiticExtraction, Signoff]
requires: [Routing, ParasiticExtraction, SPEF]
chain: Chain_STA_Basics
---
# NetDelay

## Definition
Net Delay (a.k.a. Wire Delay, Interconnect Delay, Flight Time) là thời gian tín hiệu truyền từ output pin của cell driver đến input pin của cell receiver, qua dây kim loại kết nối chúng. Net Delay phát sinh từ [[InterconnectRC]] thực tế của routed wire và [[Via]]: điện trở (R) của kim loại cản dòng, điện dung (C) của wire với substrate và coupling với wires lân cận tạo ra thời gian nạp xả. Không giống Cell Delay có thể tra LIB, Net Delay chỉ có thể tính chính xác sau khi Routing hoàn thành và Parasitic Extraction chạy xong.

## Computed from
Net Delay được tính từ RC parasitics trong SPEF file theo mô hình phân tán (Distributed RC): `t_pd ≈ 0.38 × R_total × C_total` với R_total = r × L và C_total = c × L, dẫn đến **quy luật L² (Quadratic dependence)**: `t_pd ≈ 0.38 × r × c × L²` — tức là nếu dây dài gấp đôi, Net Delay tăng gấp 4 lần, không phải 2 lần. Trong thực tế, extraction tool (Cadence QRC, Synopsys StarRC) model mỗi wire segment thành chuỗi Pi-model (C/2 — R — C/2) và dùng Elmore Delay algorithm để tính delay đến từng receiver pin riêng biệt. Ở advanced nodes (< 28nm), Coupling Capacitance giữa các wires kề nhau chiếm > 80% tổng C, làm Net Delay phụ thuộc vào switching activity của aggressor nets (Miller Effect: MCF = 0, 1, hoặc 2 tùy chiều switching).

![[Pasted image 20260521091639.png]]
## Constrains
- **[[CellDelay]]**: Output Slew của driver bị degraded qua Net (dây dài, RC lớn → sườn tín hiệu thoải hơn) → làm tăng Input Transition của receiver cell → tăng [[CellDelay]] của stage sau (Slew degradation chain)
- **[[StageDelay]]**: Net Delay là thành phần thứ hai trong Stage Delay; ở advanced nodes Net Delay thường lớn hơn [[CellDelay]] trên các critical long nets
- **[[Signoff]]**: Coupling C trong SPEF là input cho Signal Integrity analysis — Crosstalk-induced delay có thể flip timing từ Met sang Violated ở post-route signoff

## Requires
- [[Routing]] — wire geometry (length, width, layer, spacing với neighboring wires) là input bắt buộc để extract R và C; trước Routing chỉ có thể dùng Wire Load Model (WLM) ước tính — sai số lớn ở advanced nodes
- [[ParasiticExtraction]] — tool extract RC từ layout geometry + Tech LEF (layer stackup, dielectric constants); output là SPEF file
- [[SPEF]] — Standard Parasitic Exchange Format chứa R, C của từng Net segment; STA tool annotate SPEF lên timing graph để compute Net Delay chính xác

## Used by
- [[STA]] — annotate SPEF lên timing graph; Net Delay được add vào AT tại mỗi node sau khi tín hiệu rời output pin của driver
- [[ParasiticExtraction]] — tạo ra SPEF là input cho Net Delay computation; quality của SPEF quyết định độ chính xác của STA
- [[Signoff]] — post-route STA với full SPEF là lần đầu tiên Net Delay được tính chính xác; đây là lý do timing có thể fail ở signoff dù đã pass ở pre-route

## Key insight
[USER REVIEW — draft suggestion]: Net Delay là khái niệm phân biệt Physical Design engineer với logic designer — logic designer chỉ thấy [[CellDelay]] trong LIB, còn PD engineer phải đối mặt với Net Delay thực tế sau Routing. Điểm đặc biệt quan trọng: quy luật L² khiến một long wire xuyên chip có thể có Net Delay lớn hơn toàn bộ logic path từ đầu đến cuối. Giải pháp là Repeater Insertion — chèn Buffer/Inverter mỗi khoảng L_opt để phá vỡ chuỗi RC phân tán, đưa delay từ O(L²) về O(L). Đây là lý do tại sao PD engineer luôn phải quan tâm đến wire length và placement quality trước khi Routing bắt đầu.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Closely related: [[CellDelay]] · [[StageDelay]] · [[Slew]] · [[InterconnectRC]] · [[ParasiticExtraction]] · [[SPEF]]
→ Extracted by: [[ParasiticExtraction]] → stored in [[SPEF]]
→ Caused by: RC parasitics (Coupling Capacitance, Sheet Resistance)
→ Cùng nhóm: [[CellDelay]] · [[StageDelay]] · [[SetupTime]] · [[HoldTime]] · [[Slack]]