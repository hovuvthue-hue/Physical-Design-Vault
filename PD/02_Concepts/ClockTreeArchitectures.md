---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — kiến trúc clock distribution được lựa chọn dựa trên yêu cầu skew, power, routing và floorplan
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview, Floorplanning]
requires: [ClockTreeSynthesis, ClockSkew, ClockLatency, NDR]
chain: Chain_PnR_Flow
---
# ClockTreeArchitectures

## Definition
Clock tree architecture là topology tổng thể của mạng phân phối clock trên chip. Lựa chọn kiến trúc ảnh hưởng trực tiếp đến skew, insertion delay, power consumption, routing complexity, và scalability. Không có kiến trúc nào tốt nhất tuyệt đối — mỗi loại phù hợp với một tập yêu cầu thiết kế cụ thể tùy thuộc vào skew requirement, power budget, routing complexity, scalability, và performance target.

## Conventional Clock Tree (Single Source)

Dùng một single clock root source để phân phối clock qua toàn bộ chip. Clock buffers và inverters được insert theo cấu trúc phân cấp để drive tất cả sink registers.

**Đặc điểm**: phù hợp cho low- đến medium-frequency designs; power consumption thấp; CTS structure đơn giản; triển khai clock gating dễ; dễ debug và analyze.

**Hạn chế**: nhạy cảm với PVT variation; insertion delay cao cho large designs; skew variation lớn trên các long clock paths; khó scale cho very large chips.

## Clock Mesh

Dùng mạng metal dạng lưới dày đặc (dense grid-like) để phân phối clock. Nhiều clock drivers feed vào mesh đồng thời, tạo ra nhiều parallel clock propagation paths.

**Đặc điểm**: skew rất thấp; tolerant cao với OCV; lower skew sensitivity across large designs; timing robustness tốt; resistance tốt với PVT variation.

**Hạn chế**: dynamic power consumption rất cao; routing resource usage lớn; metal congestion đáng kể; triển khai clock gating khó; implementation và optimization time dài; clock capacitance cao do mesh structure.

## H-Tree Clock Structure

Kiến trúc phân phối clock đối xứng theo hình chữ H. Cấu trúc chia đệ quy clock network thành các nhánh cân bằng với equal path lengths.

**Đặc điểm**: skew thấp tự nhiên nhờ symmetric routing; OCV tolerance tốt hơn clock mesh do nhiều common paths hơn; power consumption thấp hơn clock mesh; routing resource ít hơn mesh; insertion delay predictable.

**Hạn chế**: insertion delay cao hơn clock mesh; khó triển khai cho irregular floorplans; nhạy cảm với macro blockage và placement obstacles; thường cần large/super clock buffers gần root; H-tree routes thường cần NDR và shielding để bảo vệ SI/EM.

## Fishbone Clock Tree

Dùng một main clock trunk với nhiều branch lines vuông góc tỏa ra, tạo hình dạng giống xương cá.

**Đặc điểm**: wire length ngắn hơn H-tree; insertion delay nhỏ hơn; power consumption thấp hơn; routing structure đơn giản hơn; phù hợp cho long rectangular floorplans.

**Hạn chế**: clock skew lớn hơn H-tree; routing ít symmetric hơn; nhạy cảm với placement imbalance; timing variation có thể tăng ở các sink region xa; kém scalable cho very large designs.

## Multi-Source Clock Tree (MSCTS)

Kiến trúc phân tán dùng nhiều synchronized clock injection points. Thay vì một clock root duy nhất, nhiều clock sources được đồng bộ và phân phối xuống các vùng địa lý khác nhau của chip.

**Đặc điểm**: skew thấp hơn single-source CTS nhờ clock path ngắn hơn theo vùng; OCV tolerance tốt hơn; insertion delay thấp hơn; power thấp hơn full clock mesh; flexible cho macro-dominated floorplans.

**Hạn chế**: CTS optimization phức tạp hơn; clock synchronization giữa các sources khó; clock balancing thách thức hơn; yêu cầu careful skew group management; debug và analysis khó hơn.

## Selection Trade-offs

| Tiêu chí | Conventional | Clock Mesh | H-Tree | Fishbone | MSCTS |
|---|---|---|---|---|---|
| Skew | Cao | Thấp nhất | Thấp | Trung bình | Thấp |
| Dynamic Power | Thấp | Cao nhất | Trung bình | Thấp | Trung bình |
| Routing Complexity | Thấp nhất | Cao nhất | Cao | Trung bình | Trung bình |
| OCV Tolerance | Thấp | Cao | Tốt | Trung bình | Tốt |
| Scalability | Thấp | Tốt | Trung bình | Thấp | Tốt |

## Requires
- [[ClockTreeSynthesis]] — architecture là quyết định ở đầu vào của CTS
- [[ClockSkew]] — skew target là tiêu chí chính phân biệt các architectures
- [[ClockLatency]] — insertion delay là hệ quả trực tiếp của từng architecture
- [[NDR]] — hầu hết architectures cần NDR cho clock nets, đặc biệt H-Tree

## Used by
- [[ClockTreeSynthesis]] — CTS implementation theo architecture được chọn
- [[CTSOptimization]] — optimization strategy phụ thuộc architecture
- [[CTSQualityReview]] — quality metrics phải được đọc trong context của architecture
- [[Floorplanning]] — Floorplan phải tương thích với clock architecture dự kiến

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Trade-off axes: Skew · Power · Routing Complexity · Scalability · Floorplan flexibility
→ Closely related: [[ClockTreeSynthesis]] · [[ClockSkew]] · [[ClockLatency]] · [[NDR]] · [[ClockTreeNetTypes]]
→ Cùng nhóm: [[ClockTreeSynthesis]] · [[CTSFlow]] · [[CTSOptimization]] · [[ClockSkewGroup]]