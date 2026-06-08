---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — CTS scope configuration
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview, STA]
requires: [ClockTreeSynthesis, ClockSkew, ClockLatency]
chain: Chain_PnR_Flow
---
# ClockSkewGroup

## Definition
Clock Skew Group là một tập hợp các clock sink pins được xác định để CTS cân bằng với nhau trong quá trình optimization. Các sinks trong cùng một skew group được tối ưu để đạt balanced insertion delay, balanced skew, và synchronized clock arrival time. Mặc định, tất cả synchronous clock sinks có thể thuộc về cùng một skew group.

Clocks thuộc các skew groups **khác nhau không được balanced với nhau**.

## Why Clock Skew Groups Are Important
Trong một design phức tạp, không phải tất cả clock sinks đều cần được balanced lẫn nhau:
- Các clock domains khác nhau yêu cầu skew balancing độc lập — cân bằng cross-domain không chỉ không cần thiết mà còn làm khó CTS optimization.
- Cố gắng balance tất cả sinks vào một group duy nhất làm CTS phải compromise quality của từng domain để đạt global balance.

Tách skew groups giúp:
- **Improve CTS convergence**: CTS tối ưu từng group độc lập → kết quả tốt hơn cho từng clock domain
- **Reduce unnecessary buffering**: không cần buffer để balance giữa các domains không liên quan đến nhau
- **Improve timing closure**: mỗi domain đạt skew target phù hợp với đặc thù riêng của nó
- **Reduce power**: ít buffers hơn → dynamic power thấp hơn

## Key Properties
- Mỗi skew group có CTS constraints riêng (target skew, insertion delay budget, transition target)
- Một skew group được gắn với một hoặc nhiều clock sources (root clocks)
- Sinks được assign tường minh vào skew groups phù hợp với clock domain topology của design
- Mặc định (nếu không có cấu hình riêng), tất cả synchronous sinks có thể thuộc về một default group

## Relation to CTS Objectives
ClockSkewGroup là cơ chế phạm vi hóa để CTS optimization được áp dụng đúng scope:
- [[ClockSkew]] được minimize trong nội bộ từng group
- [[ClockLatency]] được balance trong nội bộ từng group
- Giữa các groups, skew và latency không bị ràng buộc balance với nhau — đây là điểm quan trọng để tránh over-constraint

## Requires
- [[ClockTreeSynthesis]] — skew groups được configure trước khi CTS chạy và định hướng toàn bộ CTS optimization
- [[ClockSkew]] — mục tiêu chính trong nội bộ mỗi group là minimize skew
- [[ClockLatency]] — insertion delay balance là mục tiêu song song với skew

## Used by
- [[ClockTreeSynthesis]] — skew groups định nghĩa scope và constraints của CTS optimization
- [[CTSOptimization]] — optimization được thực hiện per-group, không phải global
- [[CTSQualityReview]] — skew và latency được review per-group sau CTS
- [[STA]] — post-CTS STA phản ánh kết quả balance per-group trong timing analysis

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Defined by: tập hợp clock sink pins + clock sources
→ Optimized for: balanced [[ClockSkew]] · balanced [[ClockLatency]] · synchronized clock arrival
→ Key rule: CTS chỉ balance sinks trong cùng một group; cross-group balance không được thực hiện
→ Closely related: [[ClockSkew]] · [[ClockLatency]] · [[ClockTreeSynthesis]] · [[CTSOptimization]] · [[ClockExceptions]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockLatency]] · [[ClockUncertainty]] · [[ClockTreeSynthesis]]