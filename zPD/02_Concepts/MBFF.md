---
tags: [concept, pnr-flow, placement, timing, power]
group: PnR Flow
defined_in: Post-Placement Optimization
used_by: [PreCTSOptimization, Placement, ClockTreeSynthesis, STA]
requires: [StandardCell, Placement, STA]
chain: Chain_PnR_Flow
---
# MBFF

## Definition
**MBFF (Multi-Bit Flip-Flop)** là loại sequential cell chứa nhiều bit flip-flop trong cùng một cell vật lý.

Thay vì dùng nhiều single-bit FF tách rời, design có thể dùng một MBFF để gom các bit tương thích vào cùng một phần tử.

## Banking concept
**Banking** là ý tưởng gom nhiều single-bit FF tương thích thành một MBFF.

Ví dụ minh họa: 8 FF 1-bit có thể được ánh xạ thành 1 MBFF 8-bit nếu thỏa điều kiện timing/clock/physical. Đây chỉ là ví dụ minh họa, không phải quy tắc bắt buộc cho mọi design.

## Why MBFF is used
MBFF thường được cân nhắc vì các lợi ích tiềm năng:
- nhiều bit chia sẻ clock buffer hoặc clock connection;
- giảm clock capacitance tổng ở một số trường hợp;
- có thể giảm dynamic power của clock network;
- có thể giảm area do gom cell;
- có thể rút ngắn một phần clock tree net length;
- có thể cải thiện skew/timing cục bộ khi các bit chia sẻ clock sink point chung.

Các mức cải thiện cụ thể phụ thuộc library, tool, và flow setup [Needs verification].

## Merge requirements
Không phải mọi FF đều merge được. Banking vào MBFF thường yêu cầu:
- timing constraints tương thích;
- cùng clock domain hoặc clock relation tương thích;
- tương thích về placement/locality để tránh kéo dài net bất lợi;
- có MBFF cells phù hợp trong library và được tool flow hỗ trợ [Needs verification].

## Trade-offs / risks
MBFF có thể giúp PPA, nhưng có các rủi ro cần đánh đổi:
- không phải FF nào cũng merge được;
- locality vật lý rất quan trọng, merge xa nhau có thể làm xấu data path;
- timing constraints sau merge phải tiếp tục hợp lệ;
- phụ thuộc mạnh vào library/tool support [Needs verification];
- MBFF không bảo đảm luôn cải thiện timing, power, area, hoặc skew trên mọi design.

## Position in flow
Trong ngữ cảnh PnR, MBFF thường xuất hiện ở post-placement / pre-CTS optimization để dọn QoR trước [[ClockTreeSynthesis]].

Thời điểm áp dụng chính xác tùy flow implementation của từng tool [Needs verification].

## Requires
- [[StandardCell]]
- [[Placement]]
- [[STA]]

## Used by
- [[PreCTSOptimization]]
- [[Placement]]
- [[ClockTreeSynthesis]]
- [[STA]]

## Related
- [[PreCTSOptimization]]
- [[Placement]]
- [[ClockTreeSynthesis]]
- [[STA]]
- [[Slack]]
- [[PowerOptimization]]
