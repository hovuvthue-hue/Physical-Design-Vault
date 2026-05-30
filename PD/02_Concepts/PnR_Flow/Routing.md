---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Cadence Innovus (interactive + scripted), DEF (output)
used_by: [GlobalRouting, DetailedRouting, ParasiticExtraction, STA, Signoff]
requires: [ClockTreeSynthesis, Floorplanning, LEF, DEF, SDC, MMMC]
chain: Chain_PnR_Flow
---
# Routing

## Definition
Routing là bước trong P&R flow sau [[ClockTreeSynthesis]] và trước [[ParasiticExtraction]]/[[Signoff]], nơi tool kết nối các nets bằng metal wires và vias theo connectivity của netlist. Routing biến placed/CTS database thành routed database: các đường dây/via vật lý được ghi vào route DB hoặc routed [[DEF]], tạo geometry thực tế cho extraction và signoff.

Ở mức concept, Routing không chỉ “vẽ dây” mà là bước cân bằng giữa connectivity, tài nguyên routing, timing, DRC/LVS readiness, SI, reliability và manufacturability. Các chi tiết như chiến lược route clock trước, shielding, NDR, hoặc policy sửa lỗi phụ thuộc tool/PDK/flow và cần kiểm tra theo project. [Needs verification]

## Computed from
Routing thường được nhìn như một chuỗi xử lý có kiểm soát:

- **Routing setup**: kiểm tra placed database, CTS result, clock tree topology, routing constraints, [[LEF]]/[[DEF]], [[SDC]], [[MMMC]] và technology files trước khi route.
- **[[GlobalRouting]]**: lập kế hoạch đường đi ở mức trừu tượng để ước lượng demand/supply, tránh [[CongestionAnalysis|congestion]] và hướng dẫn sử dụng layer/resource. Đây chưa phải final DRC-clean wire geometry.
- **[[DetailedRouting]]**: gán nets lên [[Track]]/metal layer cụ thể, tạo wires/vias hợp lệ theo [[RoutingGrid]], [[Pitch]] và routing rules, rồi sửa các vấn đề connectivity/geometry còn lại ở mức chi tiết.
- **Post-route handoff**: routed geometry được dùng cho [[ParasiticExtraction]], post-route STA và [[Signoff]].

## Constrains
- **Connectivity**: Routing phải hoàn tất kết nối giữa các pins của từng net; open/short là lỗi correctness cơ bản cần được loại bỏ trước handoff.
- **ParasiticExtraction**: wire/via geometry sau Routing là input để extract RC parasitics thực tế và tạo [[SPEF]]. Chất lượng route ảnh hưởng trực tiếp độ chính xác của parasitic values.
- **Timing**: actual wire delay sau Routing có thể khác estimated delay trước Routing, nên setup/hold/slew/DRV cần được re-check với extracted parasitics ở post-route STA.
- **Physical correctness**: routed layout cần sẵn sàng cho DRC/LVS và các kiểm tra Signoff liên quan; ngưỡng pass/fail cụ thể phụ thuộc rule deck, foundry và signoff policy. [Needs verification]
- **Routability / SI / reliability / manufacturability**: lựa chọn layer, spacing, via usage và detour có thể ảnh hưởng congestion, coupling noise, via reliability và DFM readiness. Các policy cụ thể cần được xác nhận theo flow. [Needs verification]

## Requires
- [[ClockTreeSynthesis]] — cung cấp clock tree topology, clock nets và vị trí clock buffer/inverter đã insert; Routing phải tôn trọng các ràng buộc này khi route phần còn lại của design.
- [[Floorplanning]] — core boundary, macro/blockage, [[RoutingBlockage]], PDN và routing resources nền tảng giới hạn không gian route khả dụng.
- [[LEF]] — định nghĩa cell abstracts, pin/obstruction shapes, [[RoutingGrid]], [[Track]], [[Pitch]], layer/via/routing rules cần cho router.
- [[DEF]] — mang placed design database và là nơi lưu/trao đổi routed geometry ở các mốc implementation.
- [[SDC]] và [[MMMC]] — timing/mode/corner context để Routing ưu tiên critical paths và chuẩn bị cho post-route timing analysis.
- Routing constraints — giới hạn layer, blockage, special-net requirement hoặc route policy; chi tiết phụ thuộc tool/flow. [Needs verification]

## Used by
- [[ParasiticExtraction]] — nhận routed wire/via geometry từ Route DB hoặc routed [[DEF]] để extract RC parasitic values và tạo [[SPEF]].
- [[STA]] (post-route) — chạy lại timing analysis với actual parasitic delays thay vì estimates; đây là checkpoint timing quan trọng trước Signoff.
- [[Signoff]] — dùng routed layout/geometry làm nền cho physical verification, timing signoff và các kiểm tra reliability/manufacturability liên quan.

## Key insight
Routing là mốc chuyển từ mô hình ước lượng sang geometry vật lý thực tế: trước Routing, net delay chủ yếu dựa trên estimate; sau Routing, wire/via geometry cho phép extraction ra parasitics thực. Vì vậy Placement và CTS quality cần được kiểm tra kỹ trước khi bước vào Routing, còn mọi sửa đổi sau route nên ưu tiên local/minimal-impact để tránh phá vỡ convergence của flow.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[ClockTreeSynthesis]] · [[Floorplanning]]
→ Internal stages: [[GlobalRouting]] · [[DetailedRouting]]
→ Downstream: [[ParasiticExtraction]] · [[STA]] · [[Signoff]]
→ Closely related: [[RoutingGrid]] · [[Track]] · [[DRC]] · [[Crosstalk]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[ClockTreeSynthesis]] · [[ParasiticExtraction]] · [[Signoff]]