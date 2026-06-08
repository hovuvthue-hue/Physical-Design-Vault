---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — CTS exception configuration
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview]
requires: [ClockTreeSynthesis, ClockSkewGroup]
chain: Chain_PnR_Flow
---
# ClockExceptions

## Definition
CTS Exceptions là các clock pins hoặc endpoints đặc biệt cần cách xử lý tùy chỉnh trong quá trình clock tree construction và optimization. CTS tool có thể không tự động nhận diện hoặc xử lý đúng các pins này như normal sink pins. Designer phải tường minh chỉ định cho CTS tool cách treat từng exception pin.

## Why CTS Exceptions Exist
Trong design phức tạp, không phải mọi clock pin đều nên được CTS xử lý như nhau:
- Một số pins là điểm phân nhánh của clock topology (ICG, clock mux) — CTS không nên insert buffer vào đó
- Một số pins không cần tham gia skew balancing nhưng clock vẫn cần đến đó
- Một số pins hoàn toàn nằm ngoài scope của CTS optimization
- Một số pins không có physical clock path nhưng cần được model cho mục đích balancing

CTS exceptions cho phép designer kiểm soát chính xác hành vi CTS tại từng loại pin đặc biệt.

## Common CTS Exception Pin Types

**Stop Pin:**
CTS dừng clock propagation tại pin này — không insert thêm clock buffer về phía downstream của pin này. Thường dùng tại input của ICG cell hoặc clock mux để giới hạn phạm vi CTS build tree.

**Nonstop Pin:**
CTS tiếp tục propagate clock qua pin này thay vì dừng lại. Thường dùng cho ICG cells hoặc generated clock structures mà CTS phải xuyên qua khi build tree. CTS treat pin này như điểm trung gian, không phải endpoint.

**Exclude Pin:**
CTS bỏ qua pin này trong quá trình skew balancing và optimization, nhưng clock network vẫn có thể propagate qua pin này. Đây là điểm phân biệt quan trọng với Ignore Pin. Dùng khi pin không cần balanced nhưng clock vẫn cần đến đó.

**Floating Pin:**
Một sink pin không có physically connected clock path thực sự. CTS có thể model estimated delay cho mục đích balancing mà không cần route thực. Thường gặp trong hierarchical design hoặc khi clock path chưa fully connected.

**Ignore Pin:**
CTS hoàn toàn bỏ qua pin này trong quá trình clock tree generation — không có skew balancing, không có clock propagation modeling.

**Through Pin:**
CTS propagate clock signal qua pin này trong khi tiếp tục build clock tree về phía downstream. Dùng khi clock path phải đi qua một cell trung gian nhưng vẫn cần tiếp tục construction.

## Key Distinction: Exclude vs Ignore

| | Exclude Pin | Ignore Pin |
|---|---|---|
| Tham gia skew balancing? | Không | Không |
| Clock network propagate qua pin? | Có | Không |
| CTS insert buffer downstream? | Có thể | Không |
| Use case | Có clock nhưng không cần balanced | Hoàn toàn ngoài scope CTS |

## Relation to ClockSkewGroup
CTS exceptions ảnh hưởng đến phạm vi của [[ClockSkewGroup]]:
- Stop/Ignore pins giới hạn phạm vi sinks trong skew group
- Nonstop/Through pins mở rộng hoặc định hướng clock propagation qua các điểm trung gian
- Exception configuration phải nhất quán với skew group definitions

## Requires
- [[ClockTreeSynthesis]] — exceptions phải được configure trước khi CTS chạy
- [[ClockSkewGroup]] — exception types ảnh hưởng đến cách sinks được phân bổ và scope của từng group

## Used by
- [[ClockTreeSynthesis]] — CTS tool đọc exception definitions và apply khi build clock tree
- [[CTSOptimization]] — optimization scope bị định hướng bởi exception configuration
- [[CTSQualityReview]] — review phải xác nhận exception handling đúng với intent của designer

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Exception types: Stop · Nonstop · Exclude · Floating · Ignore · Through
→ Key distinction: Exclude (clock propagates, no balancing) vs Ignore (completely excluded)
→ Closely related: [[ClockTreeSynthesis]] · [[ClockSkewGroup]] · [[CTSOptimization]] · [[ICG]]
→ Cùng nhóm: [[ClockTreeSynthesis]] · [[ClockSkewGroup]] · [[ICG]] · [[CTSFlow]]