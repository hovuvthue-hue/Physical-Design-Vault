---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Innovus (interactive + scripted), DEF (output)
used_by: [Routing, STA, Signoff]
requires: [Placement, SDC, LEF]
chain: Chain_PnR_Flow
---
# ClockTreeSynthesis

## Definition
Clock Tree Synthesis (CTS) là bước xây dựng **physical clock distribution network** trong PnR flow: sau [[Placement]] và trước [[Routing]]. Ở bước này, tool insert Clock Buffer/Clock Inverter giữa Clock Source và các Clock Sink để clock được phân phối thực tế thay vì ideal clock assumption.

Về nội bộ thực thi, CTS thường đi qua chuỗi [[CTSFlow]]: Clustering → Balancing → Routing Clock Tree → Post-Conditioning.

Clock Sink thường là clock pin của sequential elements (FF/latch) và có thể bao gồm clock pin của macro tùy design/library support [Needs verification].

## CTS Objectives (theo thứ tự ưu tiên)

**1. Achieve Clock Tree Targets (Highest Priority)**
- Balance clock insertion delay
- Minimize clock skew
- Minimize insertion delay

**2. Meet Clock Tree Design Rules**
- Maximum transition/fanout/capacitance
- **Minimum pulse width** — đảm bảo chiều rộng xung clock đủ lớn để các sequential elements kích hoạt đúng cách; vi phạm minimum pulse width có thể dẫn đến functional failure

**3. Meet Timing Constraints**
- Setup/Hold timing

**4. Minimize Power Consumption**
- Dynamic power (switching activity của clock buffers)
- Leakage power

**5. Minimize Noise on Clock Nets**
- Crosstalk-induced glitches
- Clock signal integrity degradation

## Computed from
CTS tool xây dựng Clock Tree theo các bước:
- **Clock tree topology**: tool chọn cấu trúc cây (H-tree, fishbone, mesh, hoặc hybrid) dựa trên phân bố vật lý của clock sinks sau Placement
- **Buffer insertion**: tool insert Clock Buffer hoặc Inverter pairs dọc theo clock tree để drive load và control Slew — số lượng và vị trí Buffer được tính từ fanout và wire
  capacitance
- **Skew balancing**: tool điều chỉnh wire length và buffer sizing để equalize arrival time tại tất cả sinks
- **Key metrics sau CTS:**
  - `[[ClockSkew]] = max(arrival time) − min(arrival time)`across all sinks trong cùng clock domain
  - `[[ClockLatency|Insertion Delay]] là độ trễ lan truyền clock từ Clock Source đến Clock Sink sau khi tree được build`
  - Cả hai phải nằm trong budget được define trong SDC

## Constrains
- **Routing**: clock nets được route với priority cao nhất và thường được assign các metal layers riêng (shielding để giảm Crosstalk) — vị trí Buffer cells insert bởi CTS tạo thêm placement constraints cho signal routing
- **Timing (Setup)**: [[ClockLatency|Insertion Delay]] tác động trực tiếp lên budget timing; post-CTS [[STA]] phải dùng giá trị propagation thực thay cho ideal clock
- **Timing (Hold)**: [[ClockSkew]] ảnh hưởng trực tiếp đến Hold Slack — Skew bất lợi có thể làm xấu [[HoldTime]]/[[Slack]] và tạo hold risk sau khi dùng propagated clock
- **Power**: Clock Tree có thể là một contributor lớn của dynamic power. Một khoảng tham khảo thô 30–40% có thể gặp trong một số practical designs, nhưng không được xem là universal rule; contribution thực tế phụ thuộc clock architecture, số sinks/flops, frequency, clock tree quality, insertion delay, buffer/inverter selection, library, và flow. [Needs verification]
- **MBFF interaction**: [[MBFF]] có thể giảm clock-sink count vì nhiều clock pins của single-bit FF được thay bằng một shared MBFF clock pin. Điều này làm thay đổi CTS sink distribution và load model, nhưng MBFF cell lớn hơn và có thể ràng buộc placement/locality; mức ảnh hưởng thực tế phụ thuộc library/tool/flow. [Needs verification]

## Tác động của CTS lên thiết kế

CTS tác động đáng kể đến chất lượng physical implementation:
- Clock buffers và inverters được insert xuyên suốt clock tree, làm tăng cell count và thay đổi design database.
- Design density tăng vì các clock cells được insert chiếm thêm placement area.
- Clock nets được fully detailed-routed bằng dedicated clock routing rules.
- Non-clock cells có thể bị relocate sang vị trí kém tối ưu hơn để nhường chỗ cho clock tree insertion và routing.
- CTS có thể introduce thêm các timing và electrical constraint violations mới.
- Aggressive CTS optimization có thể tạo ra trade-offs giữa area, power, congestion, skew, và timing quality.
## Requires
- [[Placement]] — placed database/Placement DEF cung cấp vị trí vật lý của clock sinks; đây là input bắt buộc trước CTS
- Placed netlist — connectivity sau placement để tool build clock topology đúng theo clock domains
- [[SDC]] — clock definitions, uncertainty/jitter, transition constraints và timing exceptions liên quan
- [[LEF]] + timing libraries (Liberty) — thông tin physical/timing của Clock Buffer/Inverter để chọn cell phù hợp [Needs verification]
- [[MMMC]] / RC corners — framework phân tích multi-corner cho skew/latency/transition sau CTS
- **Clock Tree Spec** — file đặc tả các mục tiêu và design-rule của clock tree, bao gồm target skew, target insertion delay, max transition, max fanout, clock buffer list, NDR rules, và shielding requirements. Đây là input chính thức bên cạnh placed database
- [[ITF]] — Interconnect Technology Files cung cấp RC parameters cho từng metal layer, dùng bởi CTS tool để ước tính wire delay trong quá trình xây dựng và cân bằng clock tree
- [[ClockSkewGroup]] — skew group definitions phải được configure trước CTS để CTS biết scope của từng optimization domain
- [[ClockExceptions]] — exception pin configurations phải được setup trước CTS để tool xử lý đúng các pins đặc biệt (Stop, Nonstop, Exclude, Ignore, Through, Floating)

## Used by
- [[Routing]] — nhận clock tree topology và vị trí buffer/inverter đã insert để triển khai clock nets vật lý
- [[STA]] (post-CTS) — dùng propagated clock với actual [[ClockSkew]] và [[ClockLatency|Insertion Delay]] thay cho ideal clock
- [[Signoff]] — chất lượng clock tree (skew/latency/transition và tác động timing/power/noise) ảnh hưởng trực tiếp tới closure

## Key insight
CTS là bước chuyển quan trọng của timing analysis: trước CTS, STA thường chạy với ideal clock assumption; sau CTS, STA phải dùng propagated clock và bắt đầu phản ánh các hiệu ứng clock network thực như [[ClockSkew]], [[ClockLatency|Insertion Delay]], và clock transition thực tế.

Vì vậy một số setup/hold risk có thể trở nên rõ hơn sau CTS, đặc biệt khi skew hoặc latency chưa cân bằng. Mức độ ảnh hưởng phụ thuộc clock topology, constraints, library, và flow cụ thể. [Needs verification]
Kết quả CTS thường cần đi qua lớp [[CTSQualityReview]] trước khi downstream closure qua [[PostCTSOptimization]], [[Routing]] và readiness cho [[Signoff]].

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Placement]]
→ Downstream: [[Routing]] · [[STA]]
→ Closely related: [[CTSFlow]] · [[CTSOptimization]] · [[ClockSkew]] · [[ClockLatency|Insertion Delay]] · [[HoldTime]] · [[Slack]] · [[PostCTSOptimization]] · [[CTSQualityReview]] · [[ClockDutyCycle]] · [[MinimumPulseWidth]] · [[ClockSkewGroup]] · [[ClockExceptions]] · [[NDR]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]]


## Liên hệ với VirtualClock và GeneratedClock
- [[VirtualClock]] không được CTS tổng hợp thành clock tree vật lý vì không có clock source vật lý trong design.
- [[GeneratedClock]] có thể cần xử lý clock tree vật lý nếu có physical source point; chi tiết phụ thuộc tool/flow. [Needs verification]
