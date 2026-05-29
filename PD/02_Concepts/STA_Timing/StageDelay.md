---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: N/A — derived concept, tính toán bởi STA tool từ CellDelay + NetDelay
used_by: [STA, LogicSynthesis, Signoff]
requires: [CellDelay, NetDelay]
chain: Chain_STA_Basics
---
# StageDelay

## Definition
Stage Delay là tổng độ trễ của một giai đoạn (stage) trong timing path, bằng tổng của [[CellDelay]] (thời gian tín hiệu truyền qua bên trong cell) và Net Delay (thời gian tín hiệu truyền qua wire đến input pin của cell tiếp theo): `Stage Delay = Cell Delay + Net Delay`. Một timing path từ Launch Flip-flop đến Capture Flip-flop bao gồm chuỗi nhiều stages liên tiếp; tổng của tất cả Stage Delays cộng với clock-to-Q delay của Launch FF tạo thành Data Arrival Time (AT) tại Endpoint.

## Computed from
`Stage_Delay(i) = Cell_Delay(i) + Net_Delay(i)` trong đó: Cell_Delay(i) được tra từ LIB LUT với Input_Transition và C_load thực tế; Net_Delay(i) được tính từ RC parasitics trong SPEF bằng Elmore Delay hoặc Pi-model. Data Arrival Time tại Endpoint: `AT = t_cq + Σ Stage_Delay(i)` với `i` chạy qua tất cả stages trong combinational logic cloud. STA tool tính AT theo Forward Propagation — cộng dồn Stage Delay từ Startpoint đến Endpoint; nếu nhiều paths hội tụ về cùng một node thì lấy MAX (worst-case cho Setup check).

## Constrains
- **[[SetupTime]]**: Tổng Stage Delays quyết định Data Arrival Time (AT); nếu AT > Required Arrival Time (RAT) → Setup violation; mọi nỗ lực optimize timing của PD engineer (cell sizing, buffer insertion, placement tightening) đều nhằm giảm AT bằng cách giảm từng Stage Delay trên critical path
- **[[HoldTime]]**: Tổng Stage Delays theo fast path (dùng [[PropagationDelay|Contamination Delay]] thay [[PropagationDelay|Propagation Delay]]) quyết định minimum AT; nếu AT_min < RAT_hold → Hold violation; đây là lý do sau CTS các short paths cần được insert delay buffers
- **[[Slack]]**: `Slack = RAT - AT` — Stage Delay là yếu tố quyết định AT, từ đó quyết định Slack của từng path

## Requires
- [[CellDelay]] — thành phần thứ nhất của Stage Delay; cần LIB file và biết Output Slew của stage trước để compute chính xác
- [[NetDelay]] — thành phần thứ hai; trước Routing dùng WLM ước tính (kém chính xác), sau Routing dùng SPEF từ ParasiticExtraction (chính xác)

## Used by
- [[STA]] — tính AT tại từng node bằng cách cộng dồn Stage Delays theo Forward Propagation; Stage Delay là building block cơ bản nhất của timing graph traversal
- [[LogicSynthesis]] — internal STA trong synthesis tool iterate over stage delays để quyết định cell selection và logic restructuring; ở pre-layout, Net Delay được ước tính bằng WLM nên Stage Delay có sai số so với post-route
- [[Signoff]] — so sánh tổng Stage Delays với clock period constraint; WNS (Worst Negative Slack) của toàn chip bị quyết định bởi Stage Delays của critical path

## Key insight
[USER REVIEW — draft suggestion]: Stage Delay là đơn vị đo lường mà PD engineer dùng hàng ngày khi debug timing violations — thay vì nhìn tổng path delay một lúc, engineer nhìn từng stage một trong timing report để tìm "điểm nghẽn" (bottleneck stage có delay bất thường lớn). Một stage có Cell Delay nhỏ nhưng Net Delay lớn → wire quá dài cần Repeater hoặc Floorplan adjustment. Một stage có Cell Delay lớn → cell yếu (drive strength thấp), load nặng → cần upsize cell hoặc reduce fanout. Skill đọc timing report và identify nguyên nhân qua Stage Delay breakdown là một trong những năng lực cốt lõi của PD engineer.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Thành phần: [[CellDelay]] + [[NetDelay]]
→ Tạo ra: Data Arrival Time → [[Slack]]
→ Được analyze bởi: [[STA]] · [[LogicSynthesis]] · [[Signoff]]
→ Cùng nhóm: [[CellDelay]] · [[NetDelay]] · [[SetupTime]] · [[HoldTime]] · [[Slack]]