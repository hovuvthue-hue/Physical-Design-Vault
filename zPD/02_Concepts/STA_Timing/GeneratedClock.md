---
tags: [concept, sta-timing, sdc, clock]
group: STA — Timing
defined_in: SDC
used_by: [STA, ClockTreeSynthesis, Signoff]
requires: [SDC, STA]
chain: Chain_STA_Basics
---
# GeneratedClock

## Definition
**GeneratedClock** là clock được suy ra từ một **Master Clock** thay vì đến trực tiếp từ clock source gốc. Nó biểu diễn các clock nội bộ được tạo bởi phần tử logic hoặc phần tử clocking trung gian.

## Master clock relationship
Generated Clock giữ quan hệ pha/tần số/timing với Master Clock ở mức mô hình phân tích. Các nguồn tạo clock thường gặp:
- divider
- PLL output
- phase-shifted clock
- gated clock / ICG output
- clock mux output

## Why GeneratedClock must be modeled
GeneratedClock cần được khai báo/mô hình rõ ràng vì:
- [[STA]] cần đúng quan hệ clock để tính setup/hold và quan hệ giữa các timing path.
- [[ClockTreeSynthesis]] cần hiểu đúng boundary clock vật lý nào phải phân phối và cân bằng trong layout.
- Mô hình sai có thể làm timing analysis sai hoặc làm clock-tree construction không phản ánh kiến trúc clock thực.

## Relation to STA
Trong [[STA]], Generated Clock giúp engine phân tích đúng clock domain dẫn xuất từ Master Clock, bao gồm tương quan arrival time và các ảnh hưởng đến [[ClockSkew]] / [[ClockLatency]].

## Relation to ClockTreeSynthesis
Với các generated clock có physical source point trong design, CTS có thể cần xử lý cây clock tương ứng để bảo đảm skew/latency ở sink thực tế. Cách xử lý cụ thể phụ thuộc kiến trúc clock và flow implementation. [Needs verification]

## Limits / tool dependence
Không phải mọi generated clock đều được tool xử lý giống nhau; hành vi phụ thuộc EDA tool, flow setup, library/PDK và định nghĩa clock architecture của dự án. [Needs verification]

## Requires
- [[SDC]]
- [[STA]]

## Used by
- [[STA]]
- [[ClockTreeSynthesis]]
- [[Signoff]]

## Related
- [[VirtualClock]]
- [[ClockSkew]]
- [[ClockLatency]]
