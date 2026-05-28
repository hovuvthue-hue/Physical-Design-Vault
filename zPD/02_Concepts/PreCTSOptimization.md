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
