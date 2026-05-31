---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Tempus (Timing) / Mentor Calibre (DRC/LVS) /
            Cadence Voltus (Power)
used_by: [Tape-out]
requires: [ParasiticExtraction, STA, Routing, ChipFinishing, DRC, DFM, GateLevelNetlist]
chain: Chain_PnR_Flow
---
# Signoff

## Definition
Signoff là giai đoạn cuối của Physical Design flow, trong đó design được verify toàn diện bằng các công cụ chuyên biệt (dedicated signoff tools) để đảm bảo đáp ứng tất cả performance specifications trước khi gửi GDS file đến foundry (Tape-out). Signoff không phải một bước đơn lẻ mà là một tập hợp các checks độc lập, mỗi check là hard gate — fail bất kỳ check nào đều block Tape-out.

## Computed from
Signoff bao gồm các checks sau, mỗi loại dùng tool riêng:

- **Timing Signoff (STA)**: chạy trên dedicated STA tool (Cadence Tempus, Synopsys PrimeTime) với SPEF thực tế ở tất cả Process Corners và Operating Conditions (voltage,
  temperature). Verify Setup Slack ≥ 0 và Hold Slack ≥ 0 trên tất cả timing paths ở tất cả corners — đây là Multi-Corner Multi-Mode (MCMM) analysis
- **Physical Verification — DRC**: kiểm tra tất cả wire geometries trong GDS tuân thủ Design Rules của foundry (min width, min spacing, via enclosure, density rules v.v.) — tool: Mentor Calibre DRC, Synopsys IC Validator
- **Physical Verification — LVS (Layout vs. Schematic)**: so sánh connectivity trong GDS layout với Gate-Level Netlist — đảm bảo layout thực tế khớp với thiết kế logic. Tool: Mentor Calibre LVS
- **Power Signoff (IR Drop / EM)**: verify voltage drop trên PDN không vượt quá IR Drop budget, và current density trong wires không gây Electromigration (EM) — tool: Cadence Voltus
- **Signal Integrity Signoff**: verify Crosstalk-induced delay và noise không gây timing violations hoặc functional failures trên critical nets

## Constrains
- **Tape-out**: Signoff là hard prerequisite — foundry không chấp nhận GDS có DRC violations hoặc LVS mismatches; Timing Signoff failure đồng nghĩa chip sẽ không hoạt động đúng spec sau khi fabricate
- **ECO loop**: nếu bất kỳ Signoff check nào fail, design phải quay lại PnR flow để fix (Engineering Change Order — ECO), sau đó re-extract và re-signoff — mỗi ECO iteration
  tốn thời gian đáng kể

## Requires
- [[ParasiticExtraction]] — SPEF là input bắt buộc cho Timing Signoff và SI Signoff; không có SPEF, không thể tính wire delay thực tế
- [[STA]] — timing analysis results từ post-route STA xác định các paths cần focus trong Timing Signoff; Signoff STA tool chạy lại độc lập với độ chính xác cao hơn PnR internal STA
- [[Routing]] — GDS layout hoàn chỉnh từ Routing là input cho DRC và LVS checks; PDN layout là input cho IR Drop/EM checks
- [[ChipFinishing]] — chuẩn bị routed layout cho final physical-verification/manufacturability readiness trước signoff handoff. [Needs verification]
- [[DRC]] — physical verification gate kiểm tra layout geometry theo rule context của process/signoff deck. [Needs verification]
- [[DFM]] — signoff-adjacent manufacturability context; DFM criteria cụ thể phụ thuộc foundry/PDK/policy. [Needs verification]
- [[GateLevelNetlist]] — Netlist sau CTS (bao gồm clock Buffer cells) là reference cho LVS comparison và cho Timing Signoff netlist

## Used by
- [[Tape-out]] — Signoff clean (tất cả checks pass) là điều kiện duy nhất để release GDS file đến foundry; không có path nào đến Tape-out mà bypass Signoff

## Key insight
[USER REVIEW — draft suggestion]:
Signoff là ranh giới giữa "thiết kế trên máy tính" và "cam kết với thực tế silicon" — sau khi GDS được gửi đến foundry, mọi thay đổi đều cực kỳ tốn kém hoặc không thể thực hiện. Điểm
quan trọng cần hiểu: Signoff tools (Tempus, Calibre) được calibrate trực tiếp với foundry process data và cho kết quả chính xác hơn PnR internal tools — đây là lý do một design có thể pass timing trong Innovus nhưng vẫn fail Tempus Signoff. Khoảng gap này gọi là "correlation gap" và là một trong những thách thức thực tế lớn nhất trong Physical Design.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[ParasiticExtraction]] · [[Routing]] · [[PostRouteOptimization]] · [[ChipFinishing]] · [[STA]]
→ Downstream: [[Tape-out]]
→ Checks bao gồm: [[DRC]] · [[LVS]] · [[IRDrop]] · [[Electromigration]] · [[SignalIntegrity]]
→ Tools: Cadence Tempus · Mentor Calibre · Cadence Voltus
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[ParasiticExtraction]]