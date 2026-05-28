---
tags: [concept, sta-timing, sdc, clock]
group: STA — Timing
defined_in: SDC
used_by: [STA, SDC, Signoff]
requires: [SDC, STA]
chain: Chain_STA_Basics
---
# VirtualClock

## Definition
**VirtualClock** là clock được khai báo trong [[SDC]] để làm mốc timing, nhưng không gắn với Clock Source vật lý (không có source pin/port thật bên trong design database).

## Why VirtualClock exists
Trong nhiều giao tiếp I/O, dữ liệu đi vào/đi ra chip cần được so sánh với một mốc clock thuộc môi trường bên ngoài. Virtual Clock giúp mô hình hóa mốc đó để [[STA]] vẫn phân tích được timing path dù clock tham chiếu không chạy trực tiếp trong netlist nội bộ.

## Relation to I/O timing
VirtualClock thường được dùng làm reference cho các ràng buộc input/output timing ở mức khái niệm:
- Input path: dữ liệu từ môi trường ngoài vào logic nội bộ được check theo mốc Virtual Clock.
- Output path: dữ liệu từ logic nội bộ ra ngoài được check theo mốc Virtual Clock.

Mục tiêu là biểu diễn đúng quan hệ thời gian giữa chip và môi trường hệ thống, thay vì chỉ nhìn clock nội bộ của chip.

## Relation to ClockTreeSynthesis
[[ClockTreeSynthesis]] chỉ tổng hợp clock tree cho các clock có nguồn vật lý trong thiết kế. Vì VirtualClock không có physical source trong design, nó **không** trở thành cây clock vật lý để CTS chèn buffer/inverter.

## Limits / tool dependence
Một số flow cho phép mô hình thêm latency/độ lệch cho Virtual Clock để phản ánh môi trường bên ngoài, nhưng cách biểu diễn chi tiết phụ thuộc tool, signoff methodology và constraint convention. [Needs verification]

## Requires
- [[SDC]]
- [[STA]]

## Used by
- [[STA]]
- [[SDC]]
- [[Signoff]]

## Related
- [[GeneratedClock]]
- [[ClockTreeSynthesis]]
