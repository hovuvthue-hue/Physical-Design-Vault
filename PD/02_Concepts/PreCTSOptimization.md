---
tags: [concept, pnr-flow, placement, timing]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [Placement, ClockTreeSynthesis, Routing, Signoff]
requires: [Placement, STA, DRVFixing, PowerOptimization]
chain: Chain_PnR_Flow
---
# PreCTSOptimization

## Definition
**PreCTSOptimization** là giai đoạn tối ưu sau [[Placement]] và trước [[ClockTreeSynthesis]].

Giai đoạn này dùng placement estimate kết hợp phản hồi từ [[STA]] để dọn QoR trước khi xây clock tree.

## Pre-CTS timing context

Trước CTS, STA vận hành với hai giả định xác định scope tối ưu:

- **Ideal clock (zero delay)**: chưa có clock tree vật lý; mọi clock sinks được giả định nhận clock edge tại cùng thời điểm → ClockSkew = 0.
- **Setup only**: chỉ setup timing được optimize. Hold không được check vì với ideal clock, hold risk chưa phản ánh thực tế — hold violations thường chỉ lộ ra sau CTS khi propagated clock được dùng.

Setup check luôn chạy ở max delay (worst corner):

$$\text{Arrival Time} = T_{ck2q} + \text{Combo Delay}$$

$$\text{Setup Slack} = T_{clk} - t_{setup} - T_{ck2q} - \text{Combo Delay}$$

Mục tiêu cụ thể:
- Reduce **WNS → 0** (fix worst setup path)
- Minimize **TNS** và **FEP** (Failing Endpoints — số endpoints vi phạm timing)
- Chuẩn bị design cho clock-aware optimization trong [[ClockTreeSynthesis]]
## Position in flow
Trong flow PnR, PreCTSOptimization nằm ở post-placement loop:
- sau khi có placed database,
- trước khi chốt handoff sang CTS.

Đây là điểm trung gian quan trọng để giảm rủi ro dồn lỗi sang các bước sau.

## What it optimizes
Ở mức high-level, PreCTSOptimization thường tập trung vào:
- **timing cleanup** (đặc biệt các path nhạy cảm trước CTS),
- **DRV fixing**,
- **power/area optimization** trong phạm vi còn giữ legality/timing,
- chuẩn bị placed database đủ tốt để chuyển sang CTS.

## Optimization methods

Sáu phương pháp cụ thể được dùng trong pre-CTS optimization, theo mức độ can thiệp tăng dần:

**1. Buffering** — chèn buffers hoặc inverters
- Fix setup violations (giảm delay bằng cách chia tải và tăng drive)
- Fix max_transition violations
- Fix max_capacitance violations

**2. Cell Sizing** — thay cell sang drive strength lớn hơn hoặc nhỏ hơn
- Upsize: fix setup violations, fix max_transition, fix max_capacitance
- Downsize: save area/power khi timing cho phép
- Cùng logic function, khác drive strength; không thay đổi connectivity

**3. VT-Swapping** — đổi cell sang Vt flavor khác (cùng footprint, không ảnh hưởng placement/routing)
- LVT → speed up → fix setup violations (pre-CTS primary use case)
- HVT → add delay → fix hold violations (áp dụng post-CTS khi hold risk lộ ra)
- HVT trên non-critical paths → power reduction khi timing slack cho phép
- Thường dùng ở "final stage" sau khi buffering/sizing chưa đủ

**4. Logic Restructuring** — thay đổi cấu trúc logic
- Gate decomposition: AOI → AND/OR/INV (giảm logic depth)
- Gate composition: AND/OR/INV → AOI (giảm cell count)
- Xóa buffers/inverters dư thừa
- Fix setup violations, area reduction

**5. Cloning** — nhân bản logic để giảm output loading
- Fix setup violations bằng cách chia fanout về hai copies
- Fix max_fanout violations

**6. Pin Swapping** — hoán đổi pins tương đương trên cell
- Đưa timing-critical net vào input path có delay nhỏ nhất → fix setup
- Đưa high-toggle net vào pin có capacitance thấp hơn → giảm dynamic power
## Relation to DRVFixing and PowerOptimization
- [[DRVFixing]] là một nhánh xử lý electrical violations trong pre-CTS cleanup.
- [[PowerOptimization]] là nhánh tối ưu power/area có kiểm soát theo timing context.

PreCTSOptimization đóng vai trò “umbrella stage”, còn DRVFixing và PowerOptimization là hai nhóm kỹ thuật trọng tâm bên trong.

## Handoff to ClockTreeSynthesis
Output mong muốn của PreCTSOptimization là trạng thái placement sẵn sàng cho [[ClockTreeSynthesis]]:
- vi phạm điện đã được xử lý ở mức chấp nhận được,
- timing posture đủ ổn định để CTS tiếp quản,
- power/tính hợp lệ vật lý không có vấn đề rõ rệt ở mức pre-CTS.

Tiêu chí cụ thể phụ thuộc flow signoff nội bộ [Needs verification].

## Requires
- [[Placement]]
- [[STA]]
- [[DRVFixing]]
- [[PowerOptimization]]

## Used by
- [[Placement]]
- [[ClockTreeSynthesis]]
- [[Routing]]
- [[Signoff]]

## Related
- [[DRVFixing]]
- [[PowerOptimization]]
- [[MBFF]]
- [[PowerAnalysis]]
- [[Placement]]
- [[STA]]
- [[ClockTreeSynthesis]]
