---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Tempus (Timing) / Mentor Calibre (DRC/LVS) /
            Cadence Voltus (Power)
used_by: []
requires: [ParasiticExtraction, STA, Routing, ChipFinishing, DRC, DFM, GateLevelNetlist]
chain: Chain_PnR_Flow
---
# Signoff

## Definition
Signoff là giai đoạn cuối của Physical Design flow, trong đó design được đánh giá bằng các signoff-oriented analyses trước khi release database sang Tape-out. Signoff không phải một bước đơn lẻ mà là một tập hợp các checks độc lập; tiêu chí pass/fail và mức độ blocking phụ thuộc foundry, PDK, tool setup và project policy. [Needs verification]

## Computed from
Signoff thường bao gồm các nhóm kiểm tra sau; tool, coverage và signoff criteria phụ thuộc methodology. [Needs verification]

- **Timing Signoff (STA)**: chạy trên dedicated STA tool (Cadence Tempus, Synopsys PrimeTime) với SPEF thực tế ở tất cả Process Corners và Operating Conditions (voltage,
  temperature). Đánh giá Setup/Hold Slack trên các analysis views được project chọn — đây là Multi-Corner Multi-Mode (MCMM) analysis. [Needs verification]
- **Physical Verification — DRC**: kiểm tra tất cả wire geometries trong GDS tuân thủ Design Rules của foundry (min width, min spacing, via enclosure, density rules v.v.) — tool: Mentor Calibre DRC, Synopsys IC Validator
- **Physical Verification — LVS (Layout vs. Schematic)**: so sánh connectivity trong GDS layout với Gate-Level Netlist — đảm bảo layout thực tế khớp với thiết kế logic. Tool: Mentor Calibre LVS
- **Power Signoff (IR Drop / EM)**: đánh giá voltage drop trên PDN và current density trong wires so với project/foundry criteria. [Needs verification] — tool: Cadence Voltus
- **Signal Integrity Signoff**: đánh giá Crosstalk-induced delay/noise trên các nets liên quan tùy SI methodology. [Needs verification]

## Constrains
- **Tape-out**: Signoff là release gate quan trọng trước khi gửi database sang foundry; DRC/LVS/timing/power/SI criteria cụ thể phụ thuộc foundry deck, PDK, methodology và project policy. [Needs verification]
- **ECO loop**: khi Signoff check chưa đạt tiêu chí, design thường cần ECO hoặc quay lại stage phù hợp trong PnR flow, sau đó re-extract/re-analyze theo phạm vi ảnh hưởng. Chi phí mỗi iteration phụ thuộc project và tool flow. [Needs verification]

## Requires
- [[ParasiticExtraction]] — SPEF là input quan trọng cho Timing/SI signoff; extraction setup và coverage phụ thuộc methodology. [Needs verification]
- [[STA]] — post-route/signoff timing analysis xác định các paths cần đánh giá; correlation giữa PnR internal STA và signoff STA phụ thuộc setup, extraction, libraries và methodology. [Needs verification]
- [[Routing]] — routed layout là input cho DRC/LVS-style physical verification và PDN-related power integrity checks. [Needs verification]
- [[ChipFinishing]] — chuẩn bị routed layout cho final physical-verification/manufacturability readiness trước signoff handoff. [Needs verification]
- [[DRC]] — physical verification gate kiểm tra layout geometry theo rule context của process/signoff deck. [Needs verification]
- [[DFM]] — signoff-adjacent manufacturability context; DFM criteria cụ thể phụ thuộc foundry/PDK/policy. [Needs verification]
- [[GateLevelNetlist]] — Netlist sau CTS (bao gồm clock Buffer cells) là reference cho LVS comparison và cho Timing Signoff netlist

## Used by
- Tape-out — Signoff-clean status là release criterion quan trọng trước khi handoff database sang foundry; policy cụ thể phụ thuộc project và foundry flow. [Needs verification]

## Key insight
Signoff là ranh giới kiểm tra cuối giữa implementation database và quyết định release sang Tape-out. Ở giai đoạn này, design được đánh giá bằng các signoff-oriented analyses như timing, physical verification, power integrity và SI tùy methodology. Một design pass trong PnR internal tools vẫn có thể cần kiểm tra lại bằng signoff tools vì correlation, rule deck, extraction setup và analysis methodology có thể khác nhau giữa các stage. Mức độ chênh lệch và tiêu chí pass/fail phụ thuộc foundry, PDK, tool setup và project policy. [Needs verification]

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[ParasiticExtraction]] · [[Routing]] · [[PostRouteOptimization]] · [[ChipFinishing]] · [[STA]]
→ Downstream: Tape-out
→ Checks bao gồm: [[DRC]] · LVS · [[IRDrop]] · [[Electromigration]] · [[SignalIntegrity]]
→ Tools: Cadence Tempus · Mentor Calibre · Cadence Voltus
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[ParasiticExtraction]]
