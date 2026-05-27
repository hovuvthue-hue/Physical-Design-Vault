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
Mục tiêu chính của TieCell là tránh cách nối trực tiếp gate input vào VDD/VSS. Ở mức khái niệm, việc đi qua TieCell giúp giữ hành vi tie-off nhất quán với cell architecture và giảm rủi ro reliability/noise theo physical design convention.

TieCell vì vậy là một phần của lớp **Physical-only Cell** phục vụ tính an toàn và tính triển khai của netlist sau khi vào PnR, không phải là cell logic xử lý hàm tổ hợp thông thường.

## Tie-high vs Tie-low
- **Tie-high**: cung cấp mức logic hằng `1`.
- **Tie-low**: cung cấp mức logic hằng `0`.

Tên cụ thể của cell (ví dụ `TIEHI*`, `TIELO*`) phụ thuộc Standard Cell Library cụ thể. **[Needs verification]**

## Computed from / inserted from
TieCell thường không xuất hiện như một logic function độc lập trong RTL; chúng được đưa vào trong giai đoạn physical implementation để phục vụ các net constant (0/1) theo policy của flow.

Thời điểm và cơ chế insertion chính xác phụ thuộc tool flow/library setup. **[Needs verification]**

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
→ Cùng nhóm: [[StandardCell]] · [[PDN]] · [[Floorplanning]]
