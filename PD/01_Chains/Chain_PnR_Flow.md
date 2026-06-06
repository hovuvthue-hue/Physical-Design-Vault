---
tags: [chain, pnr-flow]
concepts: [DesignImport, Floorplanning, CoreArea, IOCell, MacroPlacement, PlacementBlockage, RoutingBlockage, PlacementGrid, Row, RoutingGrid, Site, Track, Pitch, PDN, IRDrop, TieCell, TapCell, EndCapCell, DecapCell, FillerCell, Placement, CongestionAnalysis, PreCTSOptimization, DRVFixing, PowerAnalysis, LeakagePower, PowerOptimization, SpareCell, MBFF, ClockTreeSynthesis, CTSOptimization, CTSFlow, PostCTSOptimization, CTSQualityReview, Routing, GlobalRouting, DetailedRouting, DRC, PinAccess, SignalIntegrity, AntennaEffect, PostRouteOptimization, ChipFinishing, DFM, ParasiticExtraction, Signoff, GateLevelNetlist, PhysicalConstraints, MMMC]
---
# Chain: PnR Flow

## Mục đích
Thể hiện luồng dữ liệu và quan hệ nhân quả có hướng từ inputs của P&R tool qua các bước xử lý tuần tự đến Tape-out, trong đó output của mỗi bước là hard constraint cho bước tiếp theo.

![[Pasted image 20260521082605.png]]

## Inputs của Chain
[[DesignImport]]
[[GateLevelNetlist]] · [[SDC]] · [[LEF]] · [[DEF]] · [[MMMC]] · [[PhysicalConstraints]]

## Chuỗi nhân quả

**Bước 1 — [[DesignImport]]**

Inputs: [[GateLevelNetlist]] (Verilog, connectivity + cell list), [[LEF]] (Cell Abstracts + Tech files), [[MMMC]] (Library Sets + RC Corners + Delay Corners + Analysis Views), [[PhysicalConstraints]] (die size + IO Pad placement hoặc Block PR boundary + Pin locations + Power/Ground Net naming (khai báo VDD/VSS và global net connections)).

7 sub-steps: Netlist Import → Cell Abstracts Import → Timing Constraints Import → Physical Constraints Import → Initialize Design → Quality of Inputs Checks → Quick Timing Checks.

Exit criteria: WNS ≥ 0 và TNS ≥ 0 với zero RC delays · Timing constraint coverage = 100% · Không có Black Box · Không có errors. Nếu không pass → quay lại fix Synthesis hoặc SDC, không tiến sang Floorplanning.

**Bước 2 — [[Floorplanning]]**

Nhận: Validated design database từ DesignImport.

7 sub-steps: Define Boundary (Core Area, Aspect Ratio, Target Utilization) → Place IOs (IO Pads / Block Pins) → Place Macros (flight-line analysis, 7 key factors, FIXED status) → Define SC Areas (Pre-placed Cells: [[TapCell]] + [[EndCapCell]] + [[DecapCell]], và dùng [[FillerCell]] khi cần) → Build Power Grid ([[PDN]] rings + straps) → Check Quality (trial routing, [[IRDrop]] estimate) → Go to Placement.

Output: Core boundary, Macro FIXED positions, [[PDN]] structure, [[Row]]/[[PlacementGrid]] definitions, [[RoutingGrid]] setup, [[PlacementBlockage]], [[RoutingBlockage]]. Lưu dưới dạng [[DEF]].

Truyền xuống: Core boundary + Macro blockages + Row definitions là hard placement constraints cho Placement.

Exit: Design là routable · IR Drop budget feasible · Timing closure feasible với trial-route congestion estimate.

Key concepts trong sub-steps: [[CoreArea]] (Define Boundary) · [[IOCell]] (Place IOs) · [[MacroPlacement]] (Place Macros) · [[TapCell]] · [[EndCapCell]] · [[DecapCell]] · [[FillerCell]] (SC physical-only support) · [[PDN]] · [[IRDrop]] (power quality) · [[PlacementBlockage]] · [[RoutingBlockage]] · [[Row]] · [[PlacementGrid]] · [[RoutingGrid]] (floorplan infrastructure).

**Bước 3 — [[Placement]]**

Nhận: Core boundary + Macro FIXED positions từ Floorplanning.

Output: Cell coordinates (x,y) legalized trên Placement Grid.

Truyền xuống: Clock sink locations (x,y) là input bắt buộc cho CTS.

Exit: All cells legally placed (no overlaps, no unplaced cells) · No DRV violations (max fanout, max cap, max transition — clock net DRV được để CTS xử lý) · Low congestion (Overflow < 1%, Hotspot < 100) · No EMIR violations · Pre-route Timing feasible (no or minor timing violations).

**Bước 4 — [[ClockTreeSynthesis]]**

Nhận: Clock sink locations từ Placement.

Output: Clock tree topology, Buffer/Inverter positions, Clock net routes.

Truyền xuống: Clock tree topology và Buffer positions là fixed constraints cho Routing.

Exit: [[ClockSkew]] within budget · [[ClockLatency|Insertion Delay]] within budget · [[Slew]]/Transition clean.

**Bước 5 — [[Routing]]**

Nhận: Clock tree topology và Buffer/Inverter positions từ CTS + placed database + PDN/blockage từ Floorplanning + [[LEF]]/[[DEF]] + [[SDC]]/[[MMMC]].

Output: Routed database / routed [[DEF]] với wire/via geometries hoàn chỉnh cho signal nets và các geometry cần cho extraction/signoff; gồm [[GlobalRouting]], [[DetailedRouting]], [[PostRouteOptimization]] và handoff sang [[ChipFinishing]].

Truyền xuống: Routed wire/via geometries là input bắt buộc cho [[ParasiticExtraction]] và là layout nền cho [[Signoff]].

Exit: Connectivity complete · DRC/LVS-ready layout · No major post-route timing/routability blockers · IR Drop within budget.

**Bước 6 — [[ParasiticExtraction]]**

Nhận: Wire geometries từ Routing.

Output: [[SPEF]] — RC values thực tế của từng Net per RC Corner.

Truyền xuống: SPEF là input bắt buộc cho Timing Signoff.

Exit: SPEF complete cho tất cả corners · Coupling capacitance extracted.

**Bước 7 — [[Signoff]]**

Nhận: SPEF từ ParasiticExtraction + GDS từ Routing + Netlist.

Output: GDS release decision.

Truyền xuống: GDS file đến FAB nếu tất cả checks pass.

Exit: Timing clean (all MMMC corners × modes) · DRC clean · LVS clean · IR Drop/EM clean.

## Quan hệ giữa các bước

| From | To | Quan hệ | Dữ liệu truyền |
|---|---|---|---|
| DesignImport | Floorplanning | enables | Validated design DB; zero-RC timing clean |
| Floorplanning | Placement | hard constraint | Core boundary, Macro blockages, Row definitions, PDN |
| Placement | CTS | hard constraint | Clock sink locations (x,y) |
| CTS | Routing | hard constraint | Clock tree topology, Buffer positions |
| Routing | ParasiticExtraction | enables | Wire geometries — Route DB |
| ParasiticExtraction | Signoff | enables | SPEF — RC values per Net per Corner |
| Signoff | Tape-out | gates | GDS release decision |

## STA chạy tại 4 checkpoints trong chain này

**Post-Import STA (zero-RC):** Net Delay = 0 · Setup check với ideal clock · Dùng để verify Synthesis quality trước khi đầu tư thời gian vào Floorplan. Fail ở đây → fix Synthesis, không phải PnR.

**Post-Placement STA:** Ideal clock · Wire delay estimate (WLM) · Setup check chính · Hold với ideal clock · Dùng để verify timing feasibility trước CTS.

**Post-CTS STA (quan trọng nhất):** Propagated clock với real Skew + Insertion Delay · Hold risks có thể trở nên rõ hơn sau CTS khi propagated clock, [[ClockSkew]], và [[ClockLatency|Insertion Delay]] thực tế được đưa vào STA; mức độ phụ thuộc design/flow. [Needs verification] · Hold fix Buffers được insert tại đây trước khi Routing.

**Post-Route STA:** Full [[SPEF]] · [[NetDelay]] extracted từ actual wire geometry · Setup + Hold · Timing numbers chính xác nhất trong toàn bộ flow.

**Signoff STA:** MMMC tất cả corners × modes · Setup clean tại slow corners (dc_max) · Hold clean tại fast corners (dc_min) · Hard gate cho Tape-out.

## ECO Iteration Loop

Nếu Signoff fail, design phải quay lại bước bị ảnh hưởng để fix:

| Loại violation | Fix tại bước | Re-run từ |
|---|---|---|
| Timing violation (Setup) | Placement hoặc CTS | CTS → Routing → Extraction → Signoff |
| Timing violation (Hold) | CTS (Buffer insertion) | Routing → Extraction → Signoff |
| DRC violation | Routing | Routing → Extraction → Signoff |
| IR Drop fail | Floorplanning (PDN) | Floorplanning → toàn bộ flow |
| Congestion (routing fail) | Floorplanning (Macro placement) | Floorplanning → toàn bộ flow |

## Điểm giao quan trọng với chains khác

**Chain_STA_Basics:** [[STA]] chạy tại 5 checkpoints trong chain này (bao gồm zero-RC post-Import); [[SDC]] là input của Floorplanning và toàn bộ PnR; [[Slack]] là metric guide mọi optimization step. [[MMMC]] là framework tổ chức tất cả Analysis Views.

**Chain_LEF_to_PnR:** [[LEF]] cung cấp [[Site]]/[[Row]] definitions cho [[PlacementGrid]] và [[Track]] definitions (theo [[Pitch]]) cho [[RoutingGrid]] — điểm nối giữa LEF Geometry concepts và PnR Flow. [[CellAbstract]] là physical representation của mọi cell trong Floorplanning và Placement.

**Chain_BackEnd_Overview:** Chain này là sub-chain của Back-End Design flow tổng quát hơn — bắt đầu từ GateLevelNetlist (output của LogicSynthesis) và kết thúc bằng GDS (input của Fab).

## Concepts trong chain này
[[DesignImport]] · [[GateLevelNetlist]] · [[PhysicalConstraints]] · [[MMMC]] · [[Floorplanning]] · [[CoreArea]] · [[IOCell]] · [[MacroPlacement]] · [[TapCell]] · [[EndCapCell]] · [[DecapCell]] · [[FillerCell]] · [[PDN]] · [[IRDrop]] · [[PlacementBlockage]] · [[RoutingBlockage]] · [[Site]] · [[Row]] · [[PlacementGrid]] · [[Pitch]] · [[Track]] · [[RoutingGrid]] · [[DEF]] · [[Placement]] · [[PreCTSOptimization]] · [[DRVFixing]] · [[PowerAnalysis]] · [[LeakagePower]] · [[PowerOptimization]] · [[SpareCell]] · [[MBFF]] · [[CongestionAnalysis]] · [[ClockTreeSynthesis]] · [[CTSOptimization]] · [[CTSFlow]] · [[PostCTSOptimization]] · [[CTSQualityReview]] · [[Routing]] · [[GlobalRouting]] · [[DetailedRouting]] · [[DRC]] · [[PinAccess]] · [[SignalIntegrity]] · [[AntennaEffect]] · [[PostRouteOptimization]] · [[ChipFinishing]] · [[DFM]] · [[ParasiticExtraction]] · [[Signoff]] · [[ScanChainReordering]] · [[HFNS]]