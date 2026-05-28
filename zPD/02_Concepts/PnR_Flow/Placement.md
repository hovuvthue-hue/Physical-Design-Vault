---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Innovus (interactive + scripted), DEF (output)
used_by: [CTS, Routing, STA]
requires: [Floorplanning, GateLevelNetlist, SDC, LEF]
chain: Chain_PnR_Flow
---
# Placement

## Definition
Trong flow PnR của vault này, Placement diễn ra sau Floorplanning và trước ClockTreeSynthesis. Một số tài liệu có thể đánh số bước khác tùy phạm vi flow, nhưng graph hiện tại đặt Placement là bước sau Floorplanning.

Placement là giai đoạn legal standard-cell placement kèm pre-CTS optimization context: PnR tool xác định và tối ưu vị trí (x, y) của từng [[StandardCell]] trong Core area, sau đó đảm bảo tất cả cell được legalize trên [[PlacementGrid]] (align với [[Site]] và [[Row]]), không overlap với Macro hay cell khác. Output là placed DB (thường được lưu/trao đổi qua [[DEF]]) với cell coordinates hợp lệ.
## Computed from
Placement tool tối ưu hóa đồng thời nhiều objectives:
- **Wirelength minimization**: ước tính tổng wire length dựa trên [[HPWL]] (Half-Perimeter Wirelength) của từng Net
- **Timing-driven placement**: cells trên critical paths được đặt gần nhau để minimize wire delay, dựa trên [[Slack]] từ SDC
- **Congestion-driven placement**: phân bố cell density đều để tránh routing hotspots — tool dùng global routing estimate để dự đoán congestion
- **Utilization**: tỷ lệ (total cell area) / (Core area), được cân bằng theo mục tiêu timing/routability cụ thể của từng design [Needs verification]

Placement diễn ra theo 2 giai đoạn:
1. **[[GlobalPlacement]]**: xác định vị trí gần đúng, cho phép overlap tạm thời, tối ưu wirelength và timing
2. **[[Legalization]] + [[DetailedPlacement]]**: loại bỏ overlap, snap cells về đúng Row/Site boundaries, tinh chỉnh vị trí cuối

## Constrains
- **CTS**: vị trí vật lý của tất cả clock sinks (Flip-flop clock pins) sau Placement là input cứng để CTS tool xây dựng Clock Tree — placement quality ảnh hưởng trực tiếp đến [[ClockSkew]]
- **Routing**: [[PlacementDensity|cell density distribution]] sau Placement quyết định routing congestion — placement tệ tạo ra local congestion không thể resolve ở Routing stage
- **Timing**: wire length giữa cells trên critical paths sau Placement là yếu tố chính quyết định Setup [[Slack]] trước CTS/Routing
- **Power**: switching activity của cells gần nhau ảnh hưởng đến local IR Drop và thermal hotspots

## Requires
- [[Floorplanning]] — cung cấp Core boundary, Macro fixed positions / blockages, Row/Site/PlacementGrid infrastructure, và PDN structure đã hoàn thành
- [[GateLevelNetlist]] — danh sách tất cả cell instances và connectivity (Netlist) để tool biết cần place những gì và Net nào cần minimize wirelength
- [[SDC]] — Timing constraints để tool thực hiện timing-driven placement, ưu tiên cells trên critical paths
- [[LEF]] — Cell Abstracts cung cấp kích thước chính xác của từng Standard Cell, Pin locations, và Site/Row definitions

## Used by
- [[ClockTreeSynthesis]] — nhận vị trí chính xác của clock sinks làm input để synthesize và balance Clock Tree
- [[Routing]] — nhận vị trí cố định của tất cả cells, route signal nets giữa các Pin đã có tọa độ xác định
- [[STA]] (pre-route) — chạy timing analysis với estimated wire delays dựa trên placement để verify timing feasibility trước khi Routing
- Placement DB / [[DEF]] — chứa legalized cell locations và placement density/congestion posture để downstream steps đánh giá khả năng routability và timing trước CTS

## Key insight
[USER REVIEW — draft suggestion]:
Placement là bước quyết định chất lượng của tất cả các bước phía sau — một Placement tốt tạo ra design routable với timing closure dễ đạt; một Placement tệ tạo ra congestion và timing violations mà Routing không thể fix. Điểm quan trọng cần nhớ: sau Placement, kỹ sư đã có thể dự đoán tương đối chắc chắn liệu design có close timing được không thông qua pre-route STA và congestion map — nếu kết quả xấu ở đây, fix sớm rẻ hơn nhiều so với fix sau Routing.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Floorplanning]]
→ Downstream: [[ClockTreeSynthesis]] · [[Routing]] · [[STA]]
→ Cùng nhóm: [[Floorplanning]] · [[ClockTreeSynthesis]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]] · [[PreCTSOptimization]] · [[DRVFixing]] · [[PowerAnalysis]] · [[PowerOptimization]] · [[MBFF]]
