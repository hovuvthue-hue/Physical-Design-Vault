---
tags: [concept, cell-library, pnr-flow]
group: Cell Library
defined_in: Standard Cell Library; inserted/used during physical implementation
used_by: [Floorplanning, Placement, Optimization, PDN, Signoff]
requires: [StandardCell, PDN]
chain: Chain_LEF_to_PnR
---
# TieCell

## Definition
TieCell là một **special standard cell** dùng để cung cấp mức logic hằng cho input của logic gate theo cách an toàn ở mức physical implementation. Thay vì nối trực tiếp input vào [[PDN]] rails (VDD/VSS), flow sử dụng TieCell để tạo constant logic value qua một cell đã được library chuẩn hóa.

## Why TieCell exists
Mục tiêu chính của TieCell là thay thế direct connection từ input gate vào VDD/VSS. Gate oxide nằm dưới poly gate là phần mỏng nhất và nhạy cảm nhất của transistor — voltage surges trong quá trình fabrication ([[AntennaEffect]]) hoặc trong vận hành có thể damage gate oxide khi input pin bị nối trực tiếp vào VDD hoặc VSS. Do đó direct connection không được phép.

TieCell thay thế bằng cấu trúc diode-connected transistor: Vg = Vd, dẫn đến Vgs = Vds, và Vds > Vgs − Vt luôn thỏa — cell tạo constant output hợp lệ mà không đặt gate oxide dưới voltage stress trực tiếp.

TieCell vì vậy là một phần của lớp **Physical-only Cell** phục vụ tính an toàn vật lý của netlist sau khi vào PnR.
## Tie-high vs Tie-low
- **Tie-high**: cung cấp mức logic hằng `1` (logic high) cho các input cần constant `1`.
- **Tie-low**: cung cấp mức logic hằng `0` (logic low) cho các input cần constant `0`.

Tên cụ thể của cell (ví dụ `TIEHI*`, `TIELO*`) phụ thuộc Standard Cell Library cụ thể. **[Needs verification]**

## Computed from / inserted from
TieCell **không xuất hiện trong Gate-Level Netlist từ Logic Synthesis** — netlist sau synthesis chứa các constant literals `1'b0` và `1'b1` trực tiếp. Trong quá trình physical implementation, PnR tool thay thế toàn bộ `1'b0` bằng tie-low cells và `1'b1` bằng tie-high cells.

TieCell insertion thường diễn ra trong giai đoạn post-placement optimization; thứ tự chính xác trong flow phụ thuộc tool setup.

TieCell cũng được dùng để tie input của [[SpareCell]] về known state (VDD/VSS) để tránh floating inputs trước tape-out.

**Tie Cell Constraints:**
- **Max fanout limit:** một TieCell instance không được phép drive quá số sinks cho phép.
- **Max distance:** giới hạn khoảng cách vật lý tối đa giữa TieCell và các sinks nó phục vụ.

## Constrains
- Constant nets cần được tie-off qua TieCell thay vì nối trực tiếp input pin vào VDD/VSS rails.
- TieCell handling phải nhất quán với power/ground context của [[PDN]].
- Quy tắc fanout của mỗi TieCell instance và giới hạn deployment cụ thể là phụ thuộc library/tool. **[Needs verification]**

## Requires
- [[StandardCell]] — TieCell là thành phần thuộc thư viện cell.
- [[PDN]] — Tie-high/Tie-low hoạt động trong bối cảnh Power/Ground nets VDD/VSS.

## Used by
- [[Floorplanning]] — chuẩn bị bối cảnh PG và physical-only cell strategy mức khái niệm.
- [[Placement]] — đặt các TieCell instances vào vùng legal placement.
- [[Optimization]] — xử lý/tối ưu các constant connections trong implementation.
- [[PDN]] — liên quan trực tiếp tới power/ground connectivity semantics.
- [[Signoff]] — kiểm tra tính hợp lệ/kết nối cuối luồng.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Phân biệt với: [[TapCell]] (TapCell phục vụ well/substrate bias, không dùng để tạo logic constant)
→ Phân biệt với: [[FillerCell]] (FillerCell chủ yếu để continuity của rail/well, không tạo logic 0/1)
→ Phân biệt với: [[SpareCell]] (SpareCell là logic resource dự phòng cho ECO; thường cần TieCell để giữ input ở safe state trước khi dùng)
→ Cùng nhóm: [[StandardCell]] · [[PDN]] · [[Floorplanning]]
