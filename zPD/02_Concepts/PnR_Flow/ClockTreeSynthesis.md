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

Clock Sink thường là clock pin của sequential elements (FF/latch) và có thể bao gồm clock pin của macro tùy design/library support [Needs verification]. Mục tiêu chính là kiểm soát [[ClockSkew]], [[ClockLatency|Insertion Delay]], [[Slew]]/Transition, đồng thời giữ timing/power ở mức chấp nhận được cho post-CTS [[STA]].

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
- **Power**: Clock Tree có thể đóng góp đáng kể vào dynamic power; mức cụ thể phụ thuộc kiến trúc clock, library, và flow [Needs verification]
- **MBFF interaction**: MBFF có thể làm thay đổi cấu trúc clock sinks và clock tree load vì nhiều bits có thể chia sẻ clock connection; mức ảnh hưởng thực tế phụ thuộc library/tool/flow [Needs verification]

## Requires
- [[Placement]] — placed database/Placement DEF cung cấp vị trí vật lý của clock sinks; đây là input bắt buộc trước CTS
- Placed netlist — connectivity sau placement để tool build clock topology đúng theo clock domains
- [[SDC]] — clock definitions, uncertainty/jitter, transition constraints và timing exceptions liên quan
- [[LEF]] + timing libraries (Liberty) — thông tin physical/timing của Clock Buffer/Inverter để chọn cell phù hợp [Needs verification]
- [[MMMC]] / RC corners — framework phân tích multi-corner cho skew/latency/transition sau CTS
- CTS spec (nếu flow hỗ trợ) — mục tiêu và design-rule cho clock tree có thể được mô tả ở mức flow/tool [Needs verification]

## Used by
- [[Routing]] — nhận clock tree topology và vị trí buffer/inverter đã insert để triển khai clock nets vật lý
- [[STA]] (post-CTS) — dùng propagated clock với actual [[ClockSkew]] và [[ClockLatency|Insertion Delay]] thay cho ideal clock
- [[Signoff]] — chất lượng clock tree (skew/latency/transition và tác động timing/power/noise) ảnh hưởng trực tiếp tới closure

## Key insight
CTS là bước chuyển quan trọng của timing analysis: trước CTS, STA thường chạy với ideal clock assumption; sau CTS, STA phải dùng propagated clock và bắt đầu phản ánh các hiệu ứng clock network thực như [[ClockSkew]], [[ClockLatency|Insertion Delay]], và clock transition thực tế.

Vì vậy một số setup/hold risk có thể trở nên rõ hơn sau CTS, đặc biệt khi skew hoặc latency chưa cân bằng. Mức độ ảnh hưởng phụ thuộc clock topology, constraints, library, và flow cụ thể. [Needs verification]

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Placement]]
→ Downstream: [[Routing]] · [[STA]]
→ Closely related: [[CTSFlow]] · [[CTSOptimization]] · [[ClockSkew]] · [[ClockLatency|Insertion Delay]] · [[HoldTime]] · [[Slack]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]]
