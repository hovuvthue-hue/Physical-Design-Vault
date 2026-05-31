---
tags: [concept, pnr-flow, placement, power]
group: PnR Flow
defined_in: Placement
used_by: [Placement, PowerOptimization, PreCTSOptimization, Signoff]
requires: [StandardCell, PDN]
chain: Chain_PnR_Flow
---
# PowerAnalysis

## Definition
**PowerAnalysis** là hoạt động ước lượng và phân tích các thành phần công suất của design trong flow triển khai vật lý, đặc biệt quanh giai đoạn [[Placement]] và post-placement.

Mục tiêu chính là hiểu power đang đến từ đâu, mức độ phân bố theo block/path, và xu hướng thay đổi khi design đi từ pre-route sang post-route, để hỗ trợ quyết định tối ưu phù hợp.

## Power components
Ở mức concept, tổng công suất thường được tách thành:

**Total Power = Static Power + Dynamic Power**

Trong đó [[PowerAnalysis]] không chỉ “đo con số tổng”, mà còn phân rã thành phần để thấy trade-off giữa timing, area và power.

## Static / leakage power
**Static Power** (thường gọi là **Leakage Power**) là phần công suất tiêu tán ngay cả khi logic không switching.

Static Power / [[LeakagePower]] không phải một cơ chế đơn lẻ. Nó gồm nhiều nguồn leakage current như subthreshold leakage, gate leakage, reverse-biased junction leakage, GIDL và gate oxide leakage ở mức transistor.

Tỷ trọng tương đối phụ thuộc cell type, Vt option, process, temperature, voltage, logic state và power-analysis methodology. [Needs verification] Xem [[LeakagePower]] để biết chi tiết concept-level.

## Dynamic power
**Dynamic Power** là phần công suất liên quan đến hoạt động chuyển trạng thái của mạch khi vận hành.

Ở giai đoạn placement, dynamic power thường được ước lượng từ activity giả định hoặc activity đã annotate một phần, nên kết quả mang tính định hướng hơn là signoff-final.

## Switching vs internal power
Trong Dynamic Power có thể tách concept-level thành:
- **Switching Power**: liên quan tới nạp/xả điện dung tải (đặc biệt net/interconnect).
- **Internal Power**: tiêu tán bên trong cell khi tín hiệu chuyển trạng thái.

Phân tách này giúp định hướng tối ưu: giảm capacitance/activity có thể tác động mạnh lên switching; thay đổi cell/library có thể ảnh hưởng internal và leakage [Needs verification].

## Accuracy across implementation stages
Độ chính xác của [[PowerAnalysis]] thay đổi theo stage:
- **Placement / pre-route**: parasitics và activity chưa đầy đủ → độ chính xác hạn chế.
- **Post-route / signoff-oriented**: parasitics chi tiết hơn, activity đầy đủ hơn → kết quả đáng tin cậy hơn. Trong [[PostRouteOptimization]], bối cảnh này có thể guide power/area recovery an toàn hơn, miễn là không phá timing/DRV margin.

Vì vậy, kết quả power ở placement nên dùng để ra quyết định tối ưu sớm, không nên xem là kết luận cuối cùng cho tapeout.

## Requires
- [[StandardCell]]
- [[PDN]]

## Used by
- [[Placement]]
- [[PowerOptimization]]
- [[PreCTSOptimization]]
- [[PostRouteOptimization]]
- [[Signoff]]

## Related
- [[LeakagePower]]
- [[PowerOptimization]]
- [[PostRouteOptimization]]
- [[PDN]]
- [[IRDrop]]
- [[Placement]]
- [[Signoff]]
