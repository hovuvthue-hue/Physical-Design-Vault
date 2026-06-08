---
tags: [concept, sta-timing, clock]
group: STA — Clock
defined_in: Đặc tính vật lý đo được sau CTS — reported bởi STA tool (Tempus, PrimeTime)
used_by: [STA, ClockTreeSynthesis, Signoff]
requires: [ClockTreeSynthesis, ClockLatency]
chain: Chain_STA_Basics
---
# ClockSkew

## Definition
Clock Skew là sự chênh lệch thời gian khi cùng một tín hiệu xung nhịp đến chân clock của các Flip-flop khác nhau trên chip, do sự khác biệt vật lý về routing length và số lượng buffer trên mỗi nhánh của Clock Tree. Cụ thể: $\delta_{skew} = t_{capture\_clock} - t_{launch\_clock}$ — hiệu số giữa Clock Arrival Time tại Capture FF và Launch FF. 

Pre-CTS, clock thường được STA giả lập như **Ideal Clock** (hoặc zero-skew abstraction) để tối ưu sớm. Post-CTS, clock trở thành **Propagated Clock** và Skew được đo từ arrival-time thực tế tại các clock sinks. Positive Skew (capture clock đến muộn hơn launch clock) có thể nới Setup window nhưng thường làm Hold xấu đi; Negative Skew thì ngược lại.

![[Pasted image 20260521160507.png]]
## Computed from
Clock Skew phát sinh từ sự bất đối xứng vật lý trong Clock Tree sau [[ClockTreeSynthesis]]:

$$\delta_{skew} = \text{Latency}_{capture} - \text{Latency}_{launch}$$

Các nguyên nhân chính gây ra Clock Skew:
- **Unequal clock path delays** — sự chênh lệch tổng delay trên các nhánh khác nhau của clock tree
- **Clock buffer insertion** — số lượng và drive strength của clock buffers/inverters không đồng đều giữa các nhánh
- **Routing differences** — sự chênh lệch wire length và RC parasitics giữa các clock nets
- **Clock loading imbalance** — phân phối tải (fanout/capacitance) không đồng đều giữa các nhánh clock tree

Trong đó Latency của mỗi FF = Source Latency + Network Latency (toàn bộ delay từ clock source đến clock pin của FF đó). STA tool đo Skew bằng cách so sánh Clock Arrival Time tại từng cặp (Launch FF, Capture FF) liên quan đến cùng một timing path. Có 3 loại Skew thường gặp trong thực tế: **Local Skew** (giữa 2 FF cụ thể trên cùng một timing path — con số STA dùng để tính [[Slack]]), **Global Skew** (max − min Latency trên toàn bộ clock domain — metric đánh giá chất lượng CTS), **Useful Skew** (Skew được cố tình tạo ra để cải thiện timing trên critical paths).

## Constraints
- **[[SetupTime]]**: Positive Skew cộng thêm vào RAT_setup: $RAT = T_c + \delta_{skew} - t_{setup} - \delta_{margin}$ — có thể nới timing budget cho data path
- **[[HoldTime]]**: Positive Skew cộng thêm vào RAT_hold: $RAT_{hold} = t_{hold} + \delta_{skew}$ — thường thu hẹp Hold budget; Hold violations hay lộ ra sau CTS khi Skew thực tế xuất hiện
- **[[Slack]]**: Skew xuất hiện trong cả Setup Slack và Hold Slack equations với chiều tác động ngược nhau; vì vậy skew optimization luôn là bài toán trade-off setup/hold trong [[STA]]

## Requires
- [[ClockTreeSynthesis]] — Skew là hệ quả trực tiếp của Clock Tree structure; trước CTS, STA giả định ideal clock (Skew = 0 hoặc abstracted); sau CTS, Skew thực tế được propagated vào timing analysis
- [[ClockLatency]] — Skew được tính từ hiệu số Latency giữa hai FF; không thể có Skew nếu không có Latency per-FF

## Used by
- [[STA]] — annotate Skew vào timing graph sau CTS; dùng để adjust Setup RAT và Hold RAT trên từng timing path; Local Skew per path-pair là con số STA thực sự dùng
- [[ClockTreeSynthesis]] — Global/Local Skew là primary quality metric của CTS; CTS tool iterate để kiểm soát Skew trong khi đáp ứng Slew và Latency constraints
- [[Signoff]] — Skew tại Signoff được compute từ propagated clock với full SPEF; là thành phần của Clock Uncertainty budget verification

## Key insight
Clock Skew là khái niệm trung tâm phân biệt Pre-CTS timing và Post-CTS timing. Trước CTS, clock thường được model lý tưởng hoặc abstracted; sau CTS, STA phải dùng propagated clock với skew/latency thực tế. Skew không chỉ “xấu” tuyệt đối: trong một số flow, useful skew có thể được khai thác để hỗ trợ setup timing, nhưng luôn phải đánh đổi với hold margin và phụ thuộc tool/flow. [Needs verification]

## Related
→ Chain: [[Chain_STA_Basics]]
→ Produced by: [[ClockTreeSynthesis]]
→ Affects: [[SetupTime]] · [[HoldTime]] · [[Slack]]
→ Related: [[ClockLatency]] · [[ClockJitter]] · [[ClockUncertainty]] · [[STA]]
→ Cùng nhóm: [[ClockUncertainty]] · [[ClockLatency]] · [[ClockJitter]] · [[STA]] · [[SDC]]
