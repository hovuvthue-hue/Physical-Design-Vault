---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Placement phase — executed after Detail Placement, before Post-Placement optimization
used_by: [Placement, PreCTSOptimization, ClockTreeSynthesis, STA]
requires: [Placement, GateLevelNetlist, StandardCell]
chain: Chain_PnR_Flow
---
# HFNS

## Definition
**HFNS (High Fanout Net Synthesis)** là bước xử lý các nets có fanout quá lớn trong Placement flow bằng cách chèn buffer tree phân tán tải, đưa các nets đó về trạng thái có thể được route và tối ưu hiệu quả ở các bước downstream.

**High fanout net** là net có số sinks vượt max fanout threshold — điển hình là reset nets, scan enable, test mode control, clock gating enable, và các global control signals có thể drive hàng trăm đến hàng nghìn sinks.

## Why HFNS is needed
Nếu một net drive quá nhiều sinks mà không có buffer tree:
- Load capacitance tổng quá lớn → driver không đủ drive strength → max transition/slew DRV violations
- Net Delay cao → timing violations trên tất cả paths đi qua net đó
- Router không thể route net hiệu quả → congestion
- Pre-CTS timing estimate không chính xác → timing closure khó hơn

HFNS giải quyết những vấn đề này trước khi CTS và Routing bắt đầu — tương tự cách CTS xử lý clock nets nhưng HFNS dành cho non-clock high-fanout nets.

## What HFNS does
HFNS xác định các high-fanout nets (vượt `max_fanout` threshold từ SDC/library constraints), sau đó:
- Insert buffer tree phân cấp để chia tải
- Place các buffers gần sinks tương ứng (placement-aware)
- Giảm fanout tại từng buffer xuống mức hợp lệ
- Optimize slew và delay qua toàn bộ buffered net

## Position in Placement flow (L7 p.16)

Pre-Placement → Placement (Global Placement → Detail Placement → **HFNS** → Scan Chain Reordering) → Post-Placement

HFNS chạy sau Detail Placement để biết vị trí chính xác của sinks, từ đó đặt buffers ở vị trí optimal.

## Trade-offs
- Buffer insertion tăng cell count → area và dynamic power tăng
- Thay đổi netlist connectivity → timing graph phải được update
- Placement sau HFNS có thể cần fine-tuning để accommodate cells mới chèn vào
- HFNS có thể tương tác với CTS setup nếu net được xử lý cũng là partial clock path [Needs verification]

## Requires
- [[Placement]] — cần placed database với cell coordinates để biết sink locations
- [[GateLevelNetlist]] — cần connectivity để xác định high-fanout nets
- [[StandardCell]] — dùng buffer/inverter cells từ library

## Used by
- [[PreCTSOptimization]] — HFNS là prerequisite để pre-CTS state có clean DRV
- [[ClockTreeSynthesis]] — sau HFNS, non-clock nets đã properly buffered; CTS chỉ cần handle clock nets
- [[STA]] — post-HFNS STA phản ánh fanout-aware delay chính xác hơn

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Closely related: [[DRVFixing]] · [[Placement]] · [[PreCTSOptimization]]
→ Analogous to: [[ClockTreeSynthesis]] — nhưng cho non-clock high-fanout nets
→ Triggered by: max_fanout violations sau placement
→ Cùng nhóm: [[Placement]] · [[DRVFixing]] · [[PreCTSOptimization]] · [[MBFF]]