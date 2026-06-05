---
tags: [concept, pnr-flow, placement, dft]
group: PnR Flow
defined_in: Placement — executed after Detail Placement and HFNS
used_by: [Placement, Routing, Signoff]
requires: [Placement, GateLevelNetlist, DEF]
chain: Chain_PnR_Flow
---
# ScanChainReordering

## Definition
**Scan Chain Reordering** là quá trình sắp xếp lại thứ tự các Flip-flops trong scan chains dựa trên vị trí vật lý của chúng sau Placement, nhằm minimize wirelength giữa các scan elements và cải thiện routability.

## Background: Scan Chain là gì?

Scan Chain là chuỗi Flip-flops kết nối nối tiếp, được tạo trong giai đoạn DFT (Design for Test) để phục vụ kiểm tra sản xuất. Scan chains cho phép shift test data vào và ra khỏi chip thông qua scan input (SI) và scan output (SO) ports để phát hiện manufacturing defects.

## Why Reordering is Needed

Scan chains ban đầu được tạo ra **không có thông tin về vị trí vật lý** của từng Flip-flop (placement-unaware). Kết quả:
- Các FFs liền kề trong scan chain có thể nằm xa nhau về mặt vật lý
- Scan nets phải traverse quãng đường dài, thậm chí crisscross qua die
- Wirelength dài → routing congestion → PPA xấu

Sau khi Placement xác định vị trí vật lý, reordering sắp xếp lại FFs theo proximity.

## What Reordering Does

Tool reorders FFs trong từng scan chain dựa trên vị trí vật lý:
- Minimizes wirelength giữa consecutive scan elements
- Cải thiện routability và giảm congestion
- Preserves chain balance: số FFs per chain được giữ nguyên, chỉ thay đổi thứ tự FFs trong chain

## Position in Placement Flow

Pre-Placement → Placement (Global Placement → Detail Placement → HFNS → **Scan Chain Reordering**) → Post-Placement

Diễn ra sau [[HFNS]], khi vị trí FFs đã đủ ổn định để sắp xếp lại chain hợp lý. Scan DEF file phải được load trước khi tool có thể trace và reorder scan chains.

## Trade-offs / Notes

- Reordering thay đổi connectivity của scan nets → tool cần update design database sau bước này
- DFT thường tạo nhiều scan chains với chiều dài đồng đều; reordering preserves sự cân bằng này
- Mặc định trong một số flows: scan reorder tự động xảy ra trong placement; có thể tách thành bước riêng (place trước, reorder sau) nếu cần kiểm soát thủ công

## Benefits

- Faster test time (scan nets ngắn hơn → test shifts nhanh hơn)
- Better routability
- Improved overall PPA

## Requires
- [[Placement]] — cần placed database để biết vị trí vật lý của FFs
- [[GateLevelNetlist]] — cần connectivity để identify scan elements
- [[DEF]] — scan DEF cung cấp scan chain definitions ban đầu từ DFT flow

## Used by
- [[Placement]] — là sub-step cuối trong Placement phase trước Post-Placement
- [[Routing]] — scan nets sau reordering ngắn hơn → routing dễ hơn, ít detour hơn
- [[Signoff]] — final database phải đảm bảo scan chain connectivity hợp lệ cho DFT verification

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Adjacent step: [[HFNS]] — cùng chạy trong Placement flow
→ DFT context: scan chains là output của DFT insertion flow; ScanChainReordering là physical optimization của chúng
→ Cùng nhóm: [[HFNS]] · [[Placement]] · [[DRVFixing]] · [[PreCTSOptimization]]