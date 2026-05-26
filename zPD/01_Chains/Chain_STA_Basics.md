---
tags: [chain, sta-timing]
concepts: [CellDelay, NetDelay, StageDelay, STA, SDC, Slack, SetupTime, HoldTime, MMMC, ClockSkew, ClockUncertainty, ClockLatency, PropagationDelay]
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

[[SDC]] định nghĩa RAT thông qua: `create_clock` ($T_c$) · `set_clock_uncertainty` ($\delta_{margin}$) · `set_false_path` / `set_multicycle_path` (exceptions).

**4 loại timing paths** (xác định bởi Startpoint/Endpoint type):

| Path type | Startpoint | Endpoint | Constrained bởi |
|---|---|---|---|
| Register-to-Register | FF clock Pin | FF data Pin | `create_clock` period |
| Input-to-Register | Input Port | FF data Pin | `set_input_delay` |
| Register-to-Output | FF clock Pin | Output Port | `set_output_delay` |
| Input-to-Output | Input Port | Output Port | cả hai |

**Tầng 6 — Slack**
[[Slack]] = RAT − AT tại mỗi Endpoint.

Setup Slack:
$$\text{Slack}_{setup} = (T_c + \delta_{skew}) - (t_{cq} + t_{logic,max} + t_{setup} + \delta_{margin})$$

Hold Slack:
$$\text{Slack}_{hold} = (t_{ccq} + t_{logic,min} - \delta_{margin}) - (t_{hold} + \delta_{skew})$$

**Tầng 7 — Phán định**
- [[SetupTime]]: Setup Slack < 0 → data path quá chậm → Setup violation → có thể cứu bằng cách giảm tần số sau silicon.
- [[HoldTime]]: Hold Slack < 0 → data path quá nhanh → Hold violation → silicon kill, không có workaround.

## Hai luồng phân tích song song trong STA

**MAX path analysis — Setup Check:**
Dùng [[PropagationDelay|Propagation Delay]] ($t_{pd}$, worst-case delay) cho tất cả cells và nets. Forward Propagation lấy MAX khi nhiều paths hội tụ. Kết quả: AT_max so với RAT_setup → Setup Slack. Chạy trên Analysis View có dc_max (slow transistors + max RC).

**MIN path analysis — Hold Check:**
Dùng [[PropagationDelay|Contamination Delay]] ($t_{cd}$, best-case delay) cho tất cả cells và nets. Forward Propagation lấy MIN. Kết quả: AT_min so với RAT_hold → Hold Slack. Chạy trên Analysis View có dc_min (fast transistors + min RC).

## MMMC và Analysis Views

[[MMMC]] tổ chức không gian phân tích theo 5 lớp phân cấp: Library Set → RC Corner → Delay Corner → Constraint Mode → Analysis View. STA engine chạy đồng thời trên tất cả Active Analysis Views. Setup checks dùng View có dc_max; Hold checks dùng View có dc_min. Số lượng Analysis Views trong production design: 8–40 views.

Worst-case pairing lý thuyết: dc_max (slow transistors + hot + max RC) cho Setup, dc_min (fast transistors + cold + min RC) cho Hold. Design pass cả hai pair này sẽ pass mọi corner trung gian.

## Anatomy của Timing Report

Mỗi dòng trong timing report tương ứng một timing point (Pin của một Cell instance):
- **Fanout**: số sinks được drive — fanout lớn → high capacitive load → Slew degradation
- **Slew/Transition**: thời gian chuyển mức (10%→90% VDD); giá trị lớn → tín hiệu bị méo dạng
- **Delay**: CellDelay + NetDelay của stage này
- **Arrival Time**: accumulated delay từ Startpoint đến timing point hiện tại

Debug workflow: quét cột Delay tìm "đột biến" (stage trễ 0.5ns trong khi các stage khác 0.05ns) → kiểm tra Fanout và Slew ở đó → nếu Slew cao + Fanout lớn → ECO upsize cell đó từ X1 → X4.

## Full Clock Path Analysis (Post-CTS)

Mặc định `report_timing` ẩn Clock Tree path, chỉ tóm tắt bằng 1 dòng. Sau CTS, clock xuyên qua hàng trăm Clock Buffers — cần full clock path để phân tích Skew và CPPR. Full clock path report chia timing report thành 3 phân khu:

**Launch Clock Path:** Clock port ngoài cùng → qua Clock Buffers → đến clock Pin của launch FF.

**Data Path:** Q của launch FF → combinational logic → D của capture FF.

**Capture Clock Path:** Clock port ngoài cùng → qua Clock Buffers → đến clock Pin của capture FF.

Full clock path report là thao tác bắt buộc ở giai đoạn Post-CTS và Routing để phân tích CPPR (Clock Pessimism Removal) và [[ClockSkew]] thực tế của Clock Tree.

## Path Grouping và Path Filtering

**Path Grouping:** Mặc định STA phân nhóm paths thành 4 loại cơ sở (Reg2Reg, In2Reg, Reg2Out, In2Out). Trong designs lớn, một critical path có thể bị che khuất bởi các paths khác trong cùng nhóm có Slack âm sâu hơn — optimizer dành toàn bộ effort cho paths đó. Path Grouping cô lập một tập paths thành nhóm độc lập → optimizer phân bổ dedicated effort cho nhóm này. Ứng dụng: tạo group riêng cho paths crossing Macro boundaries (RAM-to-CPU, CPU-to-PHY).

**Path Filtering:** Với designs có hàng triệu paths, engineer dùng topological filters để trích xuất paths của một partition cụ thể:
- `-from`: chỉ định Startpoint (Input Port hoặc FF clock Pin)
- `-to`: chỉ định Endpoint (Output Port hoặc FF data Pin)
- `-through`: buộc path phải traverse qua một Net hoặc Pin trung gian cụ thể

**Hold Timing Report:** Mặc định `report_timing` check Setup. Để check Hold, dùng `-early` (Innovus/Stylus) hoặc `-delay_type min` (PrimeTime). Cấu trúc Slack đảo ngược: $\text{Slack}_{hold} = AT - RAT_{hold}$.

## Macro-metrics tổng hợp

$$WNS = \min(\text{Slack}) \text{ toàn chip} \rightarrow \text{quyết định max operating frequency}$$

$$TNS = \sum(\text{negative Slacks}) \rightarrow \text{phản ánh tổng khối lượng ECO cần giải quyết}$$

## STA chạy tại nhiều điểm trong PD flow (độ chính xác tăng dần)

**Post-Import STA (zero-RC):** Net Delay = 0 · Setup check với ideal clock · Verify Synthesis quality trước Floorplan · Accuracy thấp nhất nhưng fastest feedback · Fail ở đây → fix Synthesis, không phải PnR.

**Post-Placement STA:** Net Delay = RC estimate từ wire length (WLM) · Timing feasibility check trước CTS · Hold với ideal clock · Accuracy trung bình.

**Post-CTS STA (quan trọng nhất):** Clock = Propagated với real [[ClockSkew]] + [[ClockLatency|Insertion Delay]] · Hold violations bùng phát lần đầu · Insert Hold fix Buffers tại đây.

**Post-Route STA:** Net Delay = extracted RC từ actual wire geometry qua full [[SPEF]] · Setup + Hold · Accuracy cao nhất.

**Signoff STA — Tempus / PrimeTime:** [[MMMC]] tất cả corners × modes · Setup clean tại slow corners (dc_max) · Hold clean tại fast corners (dc_min) · Hard gate cho Tape-out.

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
| ClockUncertainty | SDC | defined in | set_clock_uncertainty → δ_margin |

## Điểm giao quan trọng với chains khác

**Chain_PnR_Flow:** [[STA]] chạy tại 5 checkpoints (Post-Import zero-RC · Post-Placement · Post-CTS · Post-Route · Signoff); [[SDC]] là input của Floorplanning và toàn bộ PnR; [[Slack]] là metric guide mọi optimization step; [[MMMC]] là framework chứa tất cả Analysis Views cho Signoff.

**Chain_LEF_to_PnR:** [[NetDelay]] phụ thuộc vào wire geometry sau Routing — geometry đó được define bởi [[LEF]] (Track, Pitch, Layer stackup); điểm nối giữa physical geometry và timing analysis.

**Chain_BackEnd_Overview:** [[SDC]] là interface chính thức giữa Front-End và Back-End; [[GateLevelNetlist]] + [[SDC]] là handoff artifacts từ Logic Synthesis sang Physical Design.

## Concepts trong chain này
[[CellDelay]] · [[NetDelay]] · [[StageDelay]] · [[STA]] · [[SDC]] · [[Slack]] · [[SetupTime]] · [[HoldTime]] · [[MMMC]] · [[ClockSkew]] · [[ClockUncertainty]] · [[ClockLatency]] · [[PropagationDelay]] · [[Slew]] · [[TimingArc]]