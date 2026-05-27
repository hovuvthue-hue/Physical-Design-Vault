---
tags: [concept, cell-library, pnr-flow, placement]
group: Cell Library
defined_in: Standard Cell Library; inserted during physical implementation / post-placement optimization
used_by: [Placement, PreCTSOptimization, Signoff]
requires: [StandardCell, TieCell]
chain: Chain_LEF_to_PnR
---
# SpareCell

## Definition
SpareCell là các instance [[StandardCell]] được pre-place trong layout để dự phòng cho thay đổi thiết kế về sau (ECO), thay vì phục vụ logic chức năng chính ngay tại thời điểm tape-out.

Ở thời điểm signoff/tape-out, SpareCell thường tồn tại như tài nguyên dự phòng và chưa tham gia active logic path của thiết kế.

## Why SpareCell exists
SpareCell tồn tại để giảm chi phí và rủi ro khi cần sửa logic muộn:
- Hỗ trợ ECO nhanh hơn, đặc biệt các trường hợp muốn tránh thay đổi lớn lên placement hiện hữu.
- Giảm nhu cầu re-placement hoặc re-route diện rộng khi phát sinh fix ở giai đoạn muộn.
- Cung cấp logic resource gần vùng cần sửa để ECO dễ triển khai hơn.

## ECO / metal-only ECO role
Khi có ECO sau signoff hoặc sau fabrication, nhóm implement có thể tận dụng SpareCell gần khu vực lỗi để nối lại bằng metal thay vì thêm mới cell ở vùng đã kín.

Mức độ áp dụng cho metal-only ECO phụ thuộc quy trình sản phẩm, lớp metal khả dụng, và policy signoff của từng công ty/tool flow. **[Needs verification]**

## Input tie-off and safe state
Để tránh floating input và switching không mong muốn, input của SpareCell thường được tie về known state thông qua [[TieCell]] (Tie-high/Tie-low).

Output của SpareCell thường để unused cho đến khi ECO thực sự cần kết nối vào mạch chức năng.

## Placement/distribution strategy
SpareCell thường được phân bố theo kiểu “rải trong core” để tăng xác suất có cell dự phòng gần vùng cần ECO.

Pattern phân bố (mật độ, khoảng cách, loại cell mix) không có một chuẩn cố định cho mọi dự án; nó phụ thuộc library, congestion posture, mục tiêu timing/power/area và kinh nghiệm flow owner. **[Needs verification]**

## Flow / library dependence
Thành phần spare (ví dụ inverter/buffer/NAND/NOR, đôi khi có cả flip-flop) và cách insert phụ thuộc vào:
- Standard cell library đang dùng,
- chiến lược ECO dự kiến,
- và khả năng automation của tool/flow cụ thể. **[Needs verification]**

## Requires
- [[StandardCell]]
- [[TieCell]]

## Used by
- [[Placement]]
- [[PreCTSOptimization]]
- [[Signoff]]

## Related
- [[TieCell]]
- [[StandardCell]]
- [[Placement]]
- [[PreCTSOptimization]]
- [[Signoff]]
