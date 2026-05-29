---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Innovus (interactive + scripted), DEF (output)
used_by: [ParasiticExtraction, STA, Signoff]
requires: [ClockTreeSynthesis, Floorplanning, LEF, SDC]
chain: Chain_PnR_Flow
---
# Routing

## Definition
Routing là bước thứ tư trong P&R flow, trong đó PnR tool kết nối tất cả các Pins của cells theo đúng connectivity trong Netlist bằng cách đặt metal wires trên các Routing Tracks được định nghĩa trong LEF. Output là Route DB (DEF với đầy đủ wire geometries) — đây là layout hoàn chỉnh đầu tiên có thể dùng để extract parasitics thực tế.

## Computed from
Routing diễn ra theo 3 giai đoạn tuần tự:

- **Global Routing**: chia chip thành các gcells (global routing cells), tìm đường đi tổng quát cho từng Net qua các gcells — mục tiêu là phân bố wire density đều, tránh congestion. Chưa có wire geometry thực tế.
- **Track Assignment**: gán từng Net vào Routing Track cụ thể trên từng metal layer — bắt đầu có wire coordinates thực tế nhưng chưa xử lý DRC.
- **Detailed Routing**: route từng Net chính xác theo Track và Via rules, đảm bảo không có DRC violations (min spacing, min width, via enclosure). Clock nets được route trước với   shielding, signal nets route sau theo Timing priority.

## Constrains
- **ParasiticExtraction**: wire geometry (length, width, layer, spacing với neighboring wires) sau Routing là input duy nhất để extract RC parasitics thực tế — chất lượng Routing quyết
  định độ chính xác của parasitic values
- **Timing**: actual wire delay sau Routing thường khác đáng kể so với estimated delay trước Routing — Setup và Hold Slack phải được re-verified với extracted parasitics (post-route STA)
- **DRC**: tất cả wire geometries phải pass Design Rule Check của foundry trước khi proceed sang Signoff — DRC violations là hard blocker cho Tape-out
- **Signal Integrity**: wire spacing quá nhỏ giữa aggressor và victim nets gây Crosstalk noise và Crosstalk-induced Delay — Routing tool phải enforce net spacing rules cho critical nets

## Requires
- [[ClockTreeSynthesis]] — clock nets và Buffer positions đã được fix; Routing nhận clock tree topology làm constraints, route clock nets trước với shielding để minimize Crosstalk
- [[Floorplanning]] — PDN (power stripes, rings) đã được route trong Floorplanning stage; signal Routing chỉ sử dụng Routing Tracks còn lại sau khi PDN và clock nets đã chiếm chỗ
- [[LEF]] — định nghĩa Routing Grid (Track pitch, direction, layer stackup), Via rules, DRC rules cho từng metal layer — Routing tool không thể hoạt động nếu thiếu LEF
- [[SDC]] — Timing constraints để tool ưu tiên route critical paths trước, enforce net topology constraints (max fanout, max capacitance)

## Used by
- [[ParasiticExtraction]] — nhận wire geometries hoàn chỉnh từ Route DB để extract RC parasitic values cho từng Net; đây là bước bắt buộc trước post-route STA
- [[STA]] (post-route) — chạy lại timing analysis với actual parasitic delays thay vì estimates; đây là lần STA chính xác nhất và là cơ sở cho Timing Signoff
- [[Signoff]] — DRC clean layout từ Routing là điều kiện cần cho Physical Verification (DRC/LVS) signoff

## Key insight
[USER REVIEW — draft suggestion]:
Routing là bước đầu tiên trong PD flow mà tất cả các con số timing trở nên thực tế — trước Routing mọi delay đều là ước tính, sau Routing mới có extracted parasitics thực sự. Hệ quả
thực tế: timing violations xuất hiện sau post-route STA thường nhiều hơn và khác về bản chất so với pre-route STA. Nếu phải fix timing violation sau Routing (ECO — Engineering Change Order), chi phí rất cao vì phải re-route nhiều nets. Đây là lý do Placement và CTS quality cần được verify kỹ trước khi bước vào Routing.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[ClockTreeSynthesis]] · [[Floorplanning]]
→ Downstream: [[ParasiticExtraction]] · [[STA]] · [[Signoff]]
→ Closely related: [[RoutingGrid]] · [[Track]] · [[DRC]] · [[Crosstalk]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[ParasiticExtraction]] · [[Signoff]]