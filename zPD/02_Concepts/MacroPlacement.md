---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Floorplanning step — sub-step 3 "Place Macros"
used_by: [Floorplanning, Placement, ClockTreeSynthesis, Routing]
requires: [HardIP, CellAbstract, CoreArea, PlacementBlockage]
chain: Chain_PnR_Flow
---
# MacroPlacement

## Definition
MacroPlacement là quá trình xác định vị trí tối ưu cho các Hard IP Macros bên trong Core Area trong giai đoạn Floorplanning. Macros — bao gồm SRAM, ROM, PLL, ADC, DAC, USB PHY, SERDES — là các khối pre-designed với fixed layout, không thể chia cắt hay thay đổi shape. Vị trí của chúng quyết định hầu hết các constraint vật lý cho tất cả các bước PnR phía sau.

**Tại sao MacroPlacement quan trọng hơn Standard Cell Placement:**

Macros chiếm diện tích lớn, tạo ra "bức tường thành" cản trở Routing Grid, và phải được FIXED (cố định vị trí) trước khi Placement tool chạy. Một Macro đặt sai vị trí không thể được optimizer tự động di chuyển — phải quay lại Floorplan.

**7 nhân tố ảnh hưởng chính đến MacroPlacement:**

| Nhân tố | Mục tiêu | Pitfall nếu vi phạm |
|---|---|---|
| 1. Timing Closure | Group macros liên quan gần nhau; favor data flow direction | Long wire delays → Setup violations không thể fix |
| 2. Power Consumption | Đặt high-toggle macros gần nhau; align với data flow | Zig-zag paths → switching capacitance cao → waste power |
| 3. System Interface | Interface macros gần related IO Pads; analog xa noisy digital | Long congested connections; noise coupling |
| 4. Clock Distribution | Clocked macros (SRAM) gần clock distribution network | Long clock routes → excessive Skew, high ClockLatency |
| 5. Routing Congestion | Adequate spacing giữa macros; tránh block routing channels | Detoured routes → timing degradation; routing failure |
| 6. Power Delivery & EM/Thermal | High-power macros gần power grid; spread out để tránh hotspot | IR Drop violations; Electromigration failures |
| 7. Placement Rules & Foundry | Giữ macro FIXED; tap cells; uniform poly orientation; distance từ die edge | Fab rejection; latch-up; Seal Ring interference |

## Computed from

**Quy trình MacroPlacement thực tế:**

Flight-line Analysis là bước tiên quyết — quan sát connectivity lines giữa các Macros và giữa Macros với IO Ports. Mục tiêu: minimize tổng wire length của flight lines và eliminate crisscross connections. Crisscross = tín hiệu phải đi vòng → wire length tăng → congestion.

**Nguyên tắc định hướng Macro (Orientation):**

Mỗi Macro có thể được rotate/flip. Quy tắc chọn orientation:
- Pins facing each other: nếu hai Macros communicate nhiều, hướng Pins của chúng về phía nhau → wire ngắn nhất
- Pins facing outward: nếu Macro chủ yếu kết nối với Standard Cells xung quanh, hướng Pins ra phía SC areas
- Pins facing IO Pads: nếu Macro interface với bên ngoài chip, hướng Pins về phía tương ứng IO Pad positions
- Pins away from corners: tránh đặt Pins ở góc Macro — routing corner rất khó

**Nguyên tắc đặt macro theo quan hệ kết nối và tài nguyên:**
- Ưu tiên đặt Macros gần IO/interface logic liên quan để giảm đường đi liên khối.
- Cân nhắc gần clock/power resources để giảm rủi ro phân phối clock và cấp nguồn.
- Chừa routing corridors đủ liên tục cho các kết nối chính giữa Macro ↔ Standard Cell areas.

**Nguyên tắc để lại Standard Cell areas tốt:**

Phần Standard Cell areas trong Core phải là các vùng rộng và liên tục (contiguous). Cần tránh:
- Chimneys: dải hẹp giữa các Macros — routing resource thấp, wire density cao
- Notch formation: góc lõm tạo bởi Macros kề nhau — tạo congestion hotspot
- Fragmented SC areas: SC area bị chia nhỏ bởi nhiều Macros — Placement tool khó optimize

Lý do: Large contiguous SC areas → better congestion control + more routing resources → less delay + less power + less area needed for timing fixes.

**Khoảng cách giữa Macros (Halo / Keep-out):**

Khoảng trống xung quanh Macro phục vụ hai mục đích: (1) buffer room cho routing channels, (2) space cho power strap VDD/VSS tapping giữa Macros. Thiếu VDD/VSS tap space giữa các Macros → Well không được connect → latch-up risk.

**Tính toán khoảng cách routing tối thiểu giữa Macros:**

$$d_{\min} = \frac{\text{Pitch of Routing Layers} \times \text{No. of Pins to be Routed}}{\text{Available Routing Layers in preferred direction}} + \text{Buffer Spacing}$$

“Đây là công thức ước lượng (heuristic) trong giai đoạn Floorplanning để dự báo khoảng cách tối thiểu giữa hai Macros; spacing thực tế vẫn phải được kiểm chứng bằng congestion analysis, trial routing, routing rules và ràng buộc cụ thể của tool/PDK. [Needs verification: chi tiết calibration theo từng flow/tool/PDK.]”

**Placement status của Macros:**

Sau khi MacroPlacement hoàn tất và được verify, engineer phải mark tất cả Macros là **FIXED** — trạng thái ngăn Placement tool di chuyển chúng trong quá trình optimization. Macro không được FIXED có thể bị Placement engine di chuyển sang vị trí suboptimal.

## Constrains
- **[[Placement]]**: Macro positions là FIXED constraints — Placement tool place Standard Cells vào các vùng còn lại sau khi Macros đã chiếm chỗ
- **[[ClockTreeSynthesis]]**: Macro positions ảnh hưởng trực tiếp đến clock tree topology; SRAM ở góc xa clock source → long clock route → large Skew
- **[[Routing]]**: Macro OBS blocks routing; Placement Blockages và Routing Blockages quanh Macros định nghĩa routing corridors

## Requires
- [[HardIP]] — Macro CellAbstract, SIZE, Pin locations
- [[CellAbstract]] — Physical representation (PR Boundary, Pins, OBS)
- [[CoreArea]] — không gian khả dụng cho Macro placement
- [[PlacementBlockage]] — Halos và blockages xung quanh Macros

## Used by
- [[Floorplanning]] — MacroPlacement là sub-step 3 trong Floorplanning flow
- [[Placement]] — Standard Cell Placement diễn ra sau khi Macros đã FIXED
- [[ClockTreeSynthesis]] — Macro positions ảnh hưởng đến clock tree topology

## Key insight
[USER REVIEW — draft suggestion]: MacroPlacement là nghệ thuật hơn là khoa học — không có một công thức toán học nào cho ra optimal solution. Engineer có kinh nghiệm dựa vào flight-line analysis + intuition về data flow để đặt Macros, sau đó chạy trial routing để verify congestion. Một heuristic đơn giản nhưng hiệu quả: "Trace the hot path first" — xác định critical path giữa Macros có WNS tệ nhất, đặt chúng gần nhau, rồi mới sắp xếp các Macros ít critical hơn. Sai lầm phổ biến nhất của người mới: chỉ focus vào timing mà quên clock distribution → SRAM bị đặt xa clock source → Clock Skew explodes sau CTS.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Analysis tool: Flight-line analysis (visualize connectivity)
→ 7 factors: Timing · Power · System Interface · Clock · Congestion · Power Delivery · Foundry Rules
→ Key techniques: Orientation analysis · Contiguous SC areas · Halo management · FIXED attribute
→ Constrains: [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]]
→ Cùng nhóm: [[Floorplanning]] · [[CoreArea]] · [[HardIP]] · [[PlacementBlockage]]