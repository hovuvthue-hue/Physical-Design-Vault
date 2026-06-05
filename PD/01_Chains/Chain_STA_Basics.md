---
tags: [chain, sta-timing]
concepts: [CellDelay, NetDelay, StageDelay, InterconnectRC, STA, SDC, Slack, SetupTime, HoldTime, MMMC, ClockSkew, ClockUncertainty, ClockLatency, VirtualClock, GeneratedClock, PropagationDelay]
---
# Chain: STA Basics

## Mục đích
Thể hiện luồng dữ liệu và quan hệ nhân quả có hướng từ các thành phần vật lý cơ bản ([[CellDelay]], [[NetDelay]]) qua quá trình phân tích timing ([[STA]]) đến kết quả phán định pass/fail ([[Slack]]), trong đó [[SDC]] là tập hợp luật lệ chi phối toàn bộ chuỗi và [[MMMC]] là framework tổ chức không gian phân tích đa chiều.

## Inputs của Chain
[[LIB]] · [[SPEF]] · [[GateLevelNetlist]] · [[SDC]] · [[MMMC]]

## Chuỗi nhân quả

**Tầng 1 — Cell timing models**
[[LIB]] cung cấp 2D LUT (Input Transition × C_load) cho từng Timing Arc của mỗi Standard Cell. Đây là nguồn dữ liệu gốc để tính [[CellDelay]].

**Tầng 2 — CellDelay**
[[CellDelay]] = f(Input Transition, C_load) — tra LUT từ LIB. Output Transition của cell trở thành Input Transition của cell tiếp theo — đây là cơ chế truyền [[Slew]] qua chuỗi stages.

**Tầng 3 — NetDelay**
[[NetDelay]] = f(R_wire, C_wire) theo mô hình phân tán:
$$t_{pd} \approx 0.38 \times r \times c \times L^2$$
RC parasitics được annotate từ [[SPEF]] sau Routing — trước đó dùng WLM ước tính. Slew degradation trên Net làm thoải sóng tín hiệu → làm tăng Input Transition của cell tiếp theo → tăng CellDelay của stage sau (feedback loop giữa NetDelay và CellDelay).

**Tầng 4 — StageDelay**
[[StageDelay]] = [[CellDelay]] + [[NetDelay]]. Data Arrival Time tích lũy:
$$AT = t_{cq} + \sum_i \text{StageDelay}_i$$

**Tầng 5 — STA**
[[STA]] nhận [[GateLevelNetlist]] (timing graph structure), [[LIB]] ([[CellDelay]]s), [[SPEF]] ([[NetDelay]]s), [[SDC]] (constraints) để chạy hai luồng song song:

**Forward Propagation:** cộng dồn Stage Delays từ Startpoints → compute AT tại mỗi node.

**Backward Propagation:** truy ngược từ Endpoints → compute RAT tại mỗi node.

[[SDC]] định nghĩa RAT thông qua clock period, generated clocks, clock uncertainty, input/output delay constraints, timing exceptions và các path constraints liên quan.

**4 loại timing paths** (xác định bởi Startpoint/Endpoint type):

| Path type | Startpoint | Endpoint | Constrained bởi |
|---|---|---|---|
| Register→Register | FF clock Pin | FF data Pin | clock constraint |
| Input→Register | Input Port | FF data Pin | input delay constraint |
| Register→Output | FF clock Pin | Output Port | output delay constraint |
| Input→Output | Input Port | Output Port | combined IO constraints |

**Tầng 6 — Slack**
[[Slack]] = RAT − AT tại mỗi Endpoint.

Setup Slack:
$$\text{Slack}_{setup} = (T_c + \delta_{skew}) - (t_{cq} + t_{logic,max} + t_{setup} + \delta_{margin})$$

Hold Slack:
$$\text{Slack}_{hold} = (t_{ccq} + t_{logic,min} - \delta_{margin}) - (t_{hold} + \delta_{skew})$$

**Tầng 7 — Phán định**
- [[SetupTime]]: Setup Slack < 0 → data path quá chậm → Setup violation là timing-closure/performance failure; trong một số bối cảnh có thể giảm operating frequency, nhưng đây không phải universal fix và phụ thuộc product requirements. [Needs verification]
- [[HoldTime]]: Hold Slack < 0 → data path quá nhanh → Hold violation là rủi ro chức năng nghiêm trọng và thường phải được clean trước Tape-out; workaround sau silicon không nên được giả định và phụ thuộc design/flow. [Needs verification]

## Hai luồng phân tích song song trong STA

**MAX path analysis — Setup Check:**
Dùng [[PropagationDelay|Propagation Delay]] ($t_{pd}$, worst-case delay) cho tất cả cells và nets. Forward Propagation lấy MAX khi nhiều paths hội tụ. Kết quả: AT_max so với RAT_setup → Setup Slack. Chạy trên Analysis View có dc_max (slow transistors + max RC).

**MIN path analysis — Hold Check:**
Dùng [[PropagationDelay|Contamination Delay]] ($t_{cd}$, best-case delay) cho tất cả cells và nets. Forward Propagation lấy MIN. Kết quả: AT_min so với RAT_hold → Hold Slack. Chạy trên Analysis View có dc_min (fast transistors + min RC).

## MMMC và Analysis Views

[[MMMC]] tổ chức không gian phân tích theo các lớp như Library Set, RC Corner, Delay Corner, Constraint Mode và Analysis View. STA engine phân tích trên các Active Analysis Views được flow chọn. Setup/Hold view selection và số lượng views trong production setup phụ thuộc product, PDK, signoff policy và methodology. [Needs verification]

Worst-case setup/hold views được dùng để cover các operating extremes quan trọng, nhưng coverage assumptions phải được verified theo MMMC setup của project. [Needs verification]

## Anatomy của Timing Report

Mỗi dòng trong timing report tương ứng một timing point (Pin của một Cell instance):
- **Fanout**: số sinks được drive — fanout lớn → high capacitive load → Slew degradation
- **Slew/Transition**: thời gian chuyển mức (10%→90% VDD); giá trị lớn → tín hiệu bị méo dạng
- **Delay**: CellDelay + NetDelay của stage này
- **Arrival Time**: accumulated delay từ Startpoint đến timing point hiện tại

Large local delay, high fanout, và poor slew có thể chỉ ra candidate cần timing/debug attention, nhưng fix đúng phụ thuộc STA context và implementation constraints. [Needs verification]

## Full Clock Path Analysis (Post-CTS)

Sau CTS, clock path nên được phân tích cùng data path vì propagated clock, [[ClockSkew]], [[ClockLatency|insertion delay]] và [[CPPR]]-related effects có thể ảnh hưởng post-CTS/post-route timing. Tool/report details là flow-specific. [Needs verification] Phân tích full clock path thường tách timing context thành 3 phần:

**Launch Clock Path:** Clock port ngoài cùng → qua Clock Buffers → đến clock Pin của launch FF.

**Data Path:** Q của launch FF → combinational logic → D của capture FF.

**Capture Clock Path:** Clock port ngoài cùng → qua Clock Buffers → đến clock Pin của capture FF.

Launch clock path, data path và capture clock path giúp nhìn rõ quan hệ giữa propagated clock, data delay, CPPR-related effects và [[ClockSkew]] trong post-CTS/post-route timing. [Needs verification]

## [[PathGrouping]] và Path Filtering

**Path Grouping:** STA có thể phân nhóm paths theo loại hoặc theo intent của flow để tách analysis/optimization groups. Cách group paths và cách optimizer dùng các groups này phụ thuộc tool setup, constraint strategy và implementation policy. [Needs verification]

**Path Filtering:** Path filtering thu hẹp analysis vào các startpoints, endpoints hoặc intermediate logic được chọn để debug một timing context cụ thể. Exact report syntax và filter semantics là tool-specific. [Needs verification]

**Hold Timing Analysis:** Hold analysis dùng minimum-delay / early conditions; setup analysis dùng maximum-delay / late conditions. Exact tool options là tool-specific. [Needs verification] Cấu trúc Slack đảo ngược: $\text{Slack}_{hold} = AT - RAT_{hold}$.

## Macro-metrics tổng hợp

$$WNS = \min(\text{Slack}) \text{ toàn chip} \rightarrow \text{quyết định max operating frequency}$$

$$TNS = \sum(\text{negative Slacks}) \rightarrow \text{phản ánh tổng khối lượng ECO cần giải quyết}$$

## STA chạy tại nhiều điểm trong PD flow (độ chính xác tăng dần)

**Post-Import STA (zero-RC):** Net Delay thường chưa có extracted RC · setup check với ideal clock · dùng để đánh giá synthesis/constraint quality trước Floorplan; độ chính xác phụ thuộc flow và model assumptions. [Needs verification]

**Post-Placement STA:** Net Delay = RC estimate từ wire length (WLM) · Timing feasibility check trước CTS · Hold với ideal clock · Accuracy trung bình.

**Post-CTS STA:** Clock được phân tích ở propagated-clock context với [[ClockSkew]] và [[ClockLatency|Insertion Delay]]; hold/setup risks có thể thay đổi khi clock network thực tế được đưa vào STA. Fix strategy phụ thuộc timing context và implementation constraints. [Needs verification]

**Post-Route STA:** Net Delay được annotate từ routed geometry qua [[SPEF]] · Setup + Hold · thường là stage timing sát layout nhất trước signoff, tùy extraction/signoff setup. [Needs verification]

**Signoff STA — Tempus / PrimeTime:** [[MMMC]] analysis trên views được project chọn · setup/hold criteria và release policy phụ thuộc signoff methodology, PDK/foundry requirements và project policy. [Needs verification]

## Quan hệ giữa các concepts

| From | To | Quan hệ | Ghi chú |
|---|---|---|---|
| LIB | CellDelay | defines | LUT (Input Transition × C_load) → delay value |
| CellDelay | NetDelay | propagates into | Output Transition → Input Transition trên Net |
| NetDelay | CellDelay | feeds back | Slew degradation → tăng CellDelay stage sau |
| CellDelay + NetDelay | StageDelay | composes | $t_{stage} = t_{cell} + t_{net}$ |
| StageDelay | STA | consumed by | Forward Propagation: $AT = t_{cq} + \sum t_{stage}$ |
| SDC | STA | constrains | Defines RAT via $T_c$, uncertainty, exceptions, path types |
| SPEF | NetDelay | annotates | RC parasitics → NetDelay chính xác sau Routing |
| MMMC | STA | organizes | Library Sets + RC Corners → Analysis Views → Active Views |
| STA | Slack | produces | $\text{Slack} = RAT - AT$ tại mỗi Endpoint |
| Slack | SetupTime | expresses | Setup Slack âm → data path quá chậm |
| Slack | HoldTime | expresses | Hold Slack âm → data path quá nhanh |
| ClockSkew | Slack | modifies | δ_skew trong Setup/Hold Slack equations |
| ClockUncertainty | SDC | defined in | clock uncertainty constraint → δ_margin |

## Điểm giao quan trọng với chains khác

**Chain_PnR_Flow:** [[STA]] chạy tại 5 checkpoints (Post-Import zero-RC · Post-Placement · Post-CTS · Post-Route · Signoff); [[SDC]] là input của Floorplanning và toàn bộ PnR; [[Slack]] là metric guide mọi optimization step; [[MMMC]] là framework chứa tất cả Analysis Views cho Signoff.

**Chain_LEF_to_PnR:** [[NetDelay]] phụ thuộc vào wire geometry sau Routing — geometry đó được define bởi [[LEF]] (Track, Pitch, Layer stackup); điểm nối giữa physical geometry và timing analysis.

**Chain_BackEnd_Overview:** [[SDC]] là interface chính thức giữa Front-End và Back-End; [[GateLevelNetlist]] + [[SDC]] là handoff artifacts từ Logic Synthesis sang Physical Design.

## Concepts trong chain này
[[CellDelay]] · [[NetDelay]] · [[StageDelay]] · [[InterconnectRC]] · [[STA]] · [[SDC]] · [[VirtualClock]] · [[GeneratedClock]] · [[Slack]] · [[SetupTime]] · [[HoldTime]] · [[MMMC]] · [[ClockSkew]] · [[ClockUncertainty]] · [[ClockLatency]] · [[PropagationDelay]] · [[Slew]] · [[TimingArc]]
