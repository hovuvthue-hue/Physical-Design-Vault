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

Ví dụ: tool có thể gom 8 FF 1-bit thành 1 MBFF 8-bit, hoặc gom 4 register banks 2-bit thành 2 register banks 4-bit — với điều kiện các registers đó có cùng timing constraints.

Về clock sink: N single-bit FF expose N clock pins độc lập cho CTS; một MBFF N-bit tương thích có thể expose một shared clock pin, nhờ đó giảm số clock sinks mà CTS nhìn thấy. Tuy nhiên MBFF cell có physical size/load lớn hơn một single FF và có thể ràng buộc placement/locality.

## Why MBFF is used
MBFF optimization reduces:
- **Area:** nhờ shared transistors và optimized transistor-level layout khi gom registers.
- **Total clock tree net length:** ít clock sink points hơn → clock tree compact hơn.
- **Number of clock tree buffers:** CTS cần ít buffers hơn khi sink count giảm.

Kết quả tổng hợp:
- Area reduction
- Power reduction
- Better clock skew control
- Timing improvement

Mức cải thiện cụ thể phụ thuộc library, tool, flow setup, và mức độ banking thực tế đạt được.

## Merge requirements
Không phải mọi FF đều merge được. Banking vào MBFF chỉ hợp lệ khi:
- các registers có **cùng timing constraints** — tool copies timing constraints sang resulting multibit register;
- tương thích về clock domain hoặc clock relation;
- tương thích về placement/locality để tránh kéo dài net bất lợi;
- library có MBFF cells phù hợp và tool flow hỗ trợ.

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
