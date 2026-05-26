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
Clock Tree Synthesis (CTS) là bước thứ ba trong P&R flow, trong đó PnR tool tự động xây dựng mạng lưới phân phối clock từ một hoặc nhiều clock sources đến tất cả clock sinks (Flip-flop clock
pins) trong design. Mục tiêu chính là minimize [[ClockSkew]] (chênh lệch arrival time giữa các sinks) và Clock Insertion Delay (thời gian từ source đến sink) trong khi đáp ứng Slew và Transition constraints. Output là CTS DB (DEF + Netlist cập nhật với clock Buffer/Inverter cells đã được insert).

## Computed from
CTS tool xây dựng Clock Tree theo các bước:
- **Clock tree topology**: tool chọn cấu trúc cây (H-tree, fishbone, mesh, hoặc hybrid) dựa trên phân bố vật lý của clock sinks sau Placement
- **Buffer insertion**: tool insert Clock Buffer hoặc Inverter pairs dọc theo clock tree để drive load và control Slew — số lượng và vị trí Buffer được tính từ fanout và wire
  capacitance
- **Skew balancing**: tool điều chỉnh wire length và buffer sizing để equalize arrival time tại tất cả sinks
- **Key metrics sau CTS:**
  - `[[ClockSkew]] = max(arrival time) − min(arrival time)`across all sinks trong cùng clock domain
  - `Insertion Delay = arrival time tại sink − launch time tại source`
  - Cả hai phải nằm trong budget được define trong SDC

## Constrains
- **Routing**: clock nets được route với priority cao nhất và thường được assign các metal layers riêng (shielding để giảm Crosstalk) — vị trí Buffer cells insert bởi CTS tạo thêm placement constraints cho signal routing
- **Timing (Setup)**: Insertion Delay làm tăng effective clock period nhìn từ góc độ data path — tool STA phải account cho giá trị này khi compute Setup Slack sau CTS
- **Timing (Hold)**: [[ClockSkew]] ảnh hưởng trực tiếp đến Hold Slack — Skew quá lớn gây Hold violations không thể fix bằng cách tăng clock period
- **Power**: Clock Tree thường chiếm 20–40% tổng dynamic power của chip — số lượng Buffer insert và clock net length quyết định clock power budget

## Requires
- [[Placement]] — vị trí chính xác (x, y) của tất cả clock sinks là input bắt buộc; CTS không thể chạy trước khi Placement hoàn thành và legalized
- [[SDC]] — clock definitions (frequency, uncertainty, Jitter), Skew budget, max Transition/Slew constraints cho clock nets, và clock groupings (multicycle, false paths)
- [[LEF]] — Cell Abstracts của Clock Buffer và Inverter cells (kích thước, drive strength, Pin locations) để tool chọn đúng cell type khi insert

## Used by
- [[Routing]] — nhận clock tree topology và Buffer positions đã fix; route clock nets với độ ưu tiên cao hơn signal nets, thường với shielding wires hai bên
- [[STA]] (post-CTS) — đây là lần STA quan trọng đầu tiên với actual [[ClockSkew]] và Insertion Delay thay vì ideal clock assumption; Hold violations thường được phát hiện lần đầu
  ở đây
- [[Signoff]] — Clock Tree quality (Skew, Insertion Delay, Transition) là một trong các criteria của Timing signoff

## Key insight
[USER REVIEW — draft suggestion]:
CTS là bước tạo ra sự chuyển đổi quan trọng trong timing analysis: trước CTS, STA chạy với "ideal clock" (zero Skew, zero Insertion Delay); sau CTS, STA phải account cho clock network thực tế. Hệ quả là Hold violations — vốn vô hình trước CTS — có thể xuất hiện hàng loạt sau CTS nếu Skew lớn. Đây là lý do Hold fixing thường được thực hiện ngay sau CTS bằng cách
insert Delay Buffers trên data paths, trước khi bước vào Routing.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Upstream: [[Placement]]
→ Downstream: [[Routing]] · [[STA]]
→ Closely related: [[ClockSkew]] · [[InsertionDelay]] · [[HoldViolation]]
→ Cùng nhóm: [[Floorplanning]] · [[Placement]] · [[Routing]] · [[ParasiticExtraction]] · [[Signoff]]
