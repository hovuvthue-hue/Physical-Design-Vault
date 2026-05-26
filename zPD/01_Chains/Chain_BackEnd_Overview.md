---
tags: [chain, backend-overview]
concepts: [LogicSynthesis, GateLevelNetlist, Floorplanning, Placement, ClockTreeSynthesis, Routing, ParasiticExtraction, Signoff]
---
# Chain: Back-End Design Overview

## Mục đích
Thể hiện bức tranh tổng quát của toàn bộ Back-End Design (Digital Implementation) flow từ RTL handoff đến Tape-out, trong đó Chain_PnR_Flow là sub-chain chi tiết của Physical Design stage.

## Inputs của Chain
[[RTL]] · [[SDC]] · [[LIB]] · [[LEF]]

## Định nghĩa ranh giới thuật ngữ

Back-End Design = Digital Implementation = Logic Synthesis + Physical Design

Physical Design = Place & Route + Parasitic Extraction + Sign-off

## Chuỗi nhân quả

**Stage 1 — Logic Synthesis**
Inputs: [[RTL]] (Verilog/VHDL) + [[SDC]] (timing constraints) + [[LIB]] (Standard Cell library).
Tool: Cadence Genus / Synopsys Design Compiler.
Process: Translation (RTL → generic Boolean) → Technology Mapping (→ Standard Cells) → Optimization (minimize PPA).
Output: [[GateLevelNetlist]] (mapped Verilog) + updated SDC.
Question answered: "Does the gate-level design meet performance specs with estimated parasitics?"

**Stage 2 — Physical Design (Place & Route)**
Inputs: [[GateLevelNetlist]] + [[SDC]] + [[LEF]] + [[LIB]].
Tool: Cadence Innovus / Synopsys ICC2.
Sub-stages: Floorplanning → Placement → CTS → Routing → Parasitic Extraction.
Output: GDS layout + updated Netlist (post-CTS) + SPEF.
Question answered: "Does the final physical design meet performance specs with actual parasitics?"
Chi tiết: xem [[Chain_PnR_Flow]].

**Stage 3 — Sign-off**
Inputs: GDS + SPEF + [[GateLevelNetlist]] + [[SDC]].
Tools: Cadence Tempus (Timing) · Mentor Calibre (DRC/LVS) · Cadence Voltus (Power).
Process: MMMC Timing Signoff + DRC + LVS + IR Drop/EM.
Output: Signoff-clean GDS.
Question answered: "Does the chip meet ALL required performance specifications under all conditions?"

**Stage 4 — Tape-out**
Output GDS gửi đến Foundry (TSMC, Samsung...) để chế tạo mask và fabricate silicon.

## Quan hệ giữa các stages

| From | To | Interface | Artifact |
|---|---|---|---|
| RTL Design | Logic Synthesis | Front-End → Back-End boundary | [[RTL]] + [[SDC]] |
| Logic Synthesis | Physical Design | Synthesis → PnR boundary | [[GateLevelNetlist]] + [[SDC]] |
| Physical Design | Sign-off | PnR → Signoff boundary | GDS + [[SPEF]] + Netlist |
| Sign-off | Tape-out | Design → Fabrication boundary | Signoff-clean GDS |

## Levels of Abstraction qua các stages

| Stage | Abstraction Level | Representation |
|---|---|---|
| RTL Design | Register-Transfer Level | Verilog modules, registers, nets |
| Logic Synthesis | Gate Level | Standard Cell instances, Netlist |
| Physical Design | Physical Level | Geometric shapes, wire coordinates |
| Sign-off | Physical Level (verified) | DRC/LVS-clean GDS |

## Timing Analysis xuyên suốt flow

STA chạy tại mỗi stage với độ chính xác tăng dần:

**Logic Synthesis STA**: Net Delay = WLM (ước tính) · Clock = Ideal · Accuracy thấp nhất · Correlation gap lớn với post-route.

**Physical Design STA** (4 checkpoints — chi tiết trong [[Chain_STA_Basics]]): Post-Placement → Post-CTS → Post-Route → Signoff.

**Signoff STA**: Full SPEF + MMMC · Accuracy cao nhất · Hard gate cho Tape-out.

## Điểm giao quan trọng với chains khác

**Chain_PnR_Flow**: Stage 2 (Physical Design) trong chain này được expanded thành toàn bộ Chain_PnR_Flow — mỗi bước trong PnR Flow là một sub-stage của Physical Design.

**Chain_STA_Basics**: [[STA]] chạy xuyên suốt toàn bộ Back-End flow; [[SDC]] là artifact kết nối tất cả stages; [[Slack]] là metric phán định pass/fail tại mọi checkpoint.

**Chain_LEF_to_PnR (chưa tạo)**: [[LEF]] là input của Physical Design stage — LEF Geometry concepts (Pitch, Track, Site, Row) là foundation của toàn bộ Physical Design.

## Concepts trong chain này
[[LogicSynthesis]] · [[GateLevelNetlist]] · [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]]