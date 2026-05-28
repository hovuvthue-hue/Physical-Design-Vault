---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: Cadence Tempus / Synopsys PrimeTime (Signoff STA) · Internal engine trong Cadence Innovus / Synopsys DC (in-design STA)
used_by: [LogicSynthesis, Placement, ClockTreeSynthesis, Routing, Signoff]
requires: [GateLevelNetlist, SDC, LIB, SPEF]
chain: Chain_STA_Basics
---
# STA

## Definition
Static Timing Analysis (STA) là phương pháp xác minh timing của mạch kỹ thuật số bằng cách cộng dồn [[CellDelay]] và [[NetDelay]] dọc theo tất cả timing paths trong design, sau đó so sánh với timing constraints — mà không cần simulate chức năng logic (không cần test vectors). STA model toàn bộ design dưới dạng một timing graph có hướng (directed graph): nodes là pins/ports, edges là timing arcs với weight là delay. Từ graph này, tool tính Arrival Time (AT) tại mọi node bằng Forward Propagation và Required Arrival Time (RAT) bằng Backward Propagation, rồi compute Slack = RAT − AT tại mỗi Endpoint để phán định pass/fail.

![[Pasted image 20260426222933.png]]
## Computed from
STA hoạt động qua 4 bước cốt lõi:

- **Timing graph construction**: Build directed graph từ GateLevelNetlist — nodes là input/output pins của mỗi cell và port; edges là cell timing arcs (từ LIB) và net arcs (từ SPEF hoặc WLM)
- **Forward Propagation (AT computation)**: Quét từ Startpoints (clock pins của Launch FFs, primary inputs) theo chiều thuận; tại mỗi node cộng delay của edge vào AT của node trước; nếu nhiều paths hội tụ → lấy MAX (worst-case cho Setup), lấy MIN (best-case cho Hold)
- **Backward Propagation (RAT computation)**: Quét từ Endpoints (data pins của Capture FFs, primary outputs) ngược chiều; RAT tại mỗi node = RAT của Endpoint − delay còn lại trên path; nếu nhiều paths rẽ từ một node → lấy MIN (khắt khe nhất)
- **Slack computation**: `Setup Slack = RAT(node) − AT(node)`; `Hold Slack = AT(node) − RAT(node)`; WNS = min(Slack) toàn chip; TNS = sum(negative Slacks)

STA chạy tại nhiều thời điểm trong PD flow với độ chính xác tăng dần: pre-layout (WLM → ước tính), post-placement (RC estimate từ wire length), post-CTS (propagated clock tree, real [[ClockSkew]] và [[ClockLatency]]), post-route (full SPEF → chính xác nhất).

## Constrains
- **[[LogicSynthesis]]**: Internal STA engine trong synthesis tool guide cell selection và logic optimization; quality của synthesis SDC constraints quyết định chất lượng Netlist đưa vào PnR
- **[[Placement]]**: Timing-driven placement dùng STA results (Slack per path) để ưu tiên đặt cells trên critical paths gần nhau; pre-route STA sau Placement là checkpoint quan trọng — nếu WNS quá âm ở đây, không thể fix bằng Routing
- **[[ClockTreeSynthesis]]**: STA chạy lại sau CTS với propagated clock tree thay cho ideal clock assumption; skew/latency thực tế và nhiều hold risks thường lộ rõ ở giai đoạn này
- **[[Routing]]**: Post-route STA với full SPEF là lần chính xác nhất; Net Delay thực tế thường khác ước tính → timing có thể flip từ Met sang Violated
- **[[Signoff]]**: Signoff STA tool (Tempus, PrimeTime) chạy độc lập với thuật toán khắt khe hơn in-design engine; MCMM analysis verify tất cả corners × modes; kết quả là gating criterion cho Tape-out

## Requires
- [[GateLevelNetlist]] — định nghĩa connectivity của timing graph: cell instances, nets, pin connections; không có Netlist, STA không biết tín hiệu đi từ đâu đến đâu
- [[SDC]] — Timing constraints: clock definitions (create_clock → set clock period), I/O delays, timing exceptions (false path, multicycle path); SDC là "luật lệ" mà STA dùng để compute RAT và phán định pass/fail; SDC sai → STA results vô nghĩa (Garbage In, Garbage Out)
- [[LIB]] — Cell timing models (NLDM LUT hoặc CCS current waveforms) cho từng Timing Arc; LIB characterize ở specific PVT corner; MMMC STA cần nhiều LIB files cho nhiều corners
- [[SPEF]] — Post-route parasitic data (R, C per net segment); bắt buộc cho post-route STA và Signoff; trước Routing thay bằng Wire Load Model (WLM) — kém chính xác ở advanced nodes

## Used by
- [[LogicSynthesis]] — in-design STA guide optimization loops; synthesis tool chạy STA sau mỗi pass để verify timing constraints được đáp ứng trước khi output Netlist
- [[Placement]] — timing-driven placement dùng STA Slack per path để weight placement cost function; cells có Slack âm được ưu tiên đặt gần nhau để rút ngắn wire length → giảm Net Delay
- [[ClockTreeSynthesis]] — STA sau CTS (post-CTS STA) là milestone quan trọng: dùng propagated clock với real [[ClockSkew]] và [[ClockLatency|Insertion Delay]] thay vì ideal clock; nhiều hold violations phát sinh ở đây cần fix trước Routing
- [[Routing]] — timing-driven routing dùng STA Slack để ưu tiên route critical paths trên lower-resistance layers; post-route STA với full SPEF là final check trước Signoff
- [[Signoff]] — Timing Signoff = STA pass tại tất cả MMMC views (Setup clean ở slow corners, Hold clean ở fast corners); là hard prerequisite cho Tape-out

## Key insight
[USER REVIEW — draft suggestion]: STA có 3 đặc điểm làm nó trở thành tiêu chuẩn sign-off của toàn ngành: (1) Exhaustive — duyệt 100% paths thay vì chỉ paths mà test vectors kích hoạt; (2) Fast — không simulate logic nên chạy nhanh hơn gate-level simulation hàng nghìn lần; (3) Deterministic — cùng inputs luôn cho cùng kết quả. Nhưng STA có một điểm mù cực kỳ nguy hiểm: nó tin tưởng tuyệt đối vào SDC constraints. Một false path được khai báo sai sẽ khiến STA bỏ qua violation thực sự, dẫn đến silicon failure — và STA sẽ không cảnh báo gì cả. Đây là lý do constraints validation (check_timing, report_analysis_coverage) là bước bắt buộc trước mỗi STA run.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Inputs: [[GateLevelNetlist]] · [[SDC]] · [[LIB]] · [[SPEF]]
→ Core metrics: [[Slack]] · [[SetupTime]] · [[HoldTime]] · [[ClockSkew]]
→ Building blocks: [[CellDelay]] · [[NetDelay]] · [[StageDelay]]
→ Chạy tại: [[LogicSynthesis]] → [[Placement]] → [[ClockTreeSynthesis]] → [[Routing]] → [[Signoff]]
→ Signoff tools: Cadence Tempus · Synopsys PrimeTime
→ Cùng nhóm: [[CellDelay]] · [[NetDelay]] · [[StageDelay]] · [[SetupTime]] · [[HoldTime]] · [[Slack]] · [[SDC]]