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
Clock Skew là sự chênh lệch thời gian khi cùng một tín hiệu xung nhịp đến chân clock của các Flip-flop khác nhau trên chip, do sự khác biệt vật lý về routing length và số lượng buffer trên mỗi nhánh của Clock Tree. Cụ thể: $\delta_{skew} = t_{capture\_clock} - t_{launch\_clock}$ — hiệu số giữa Clock Arrival Time tại Capture FF và Launch FF. Positive Skew (capture clock đến muộn hơn launch clock) nới rộng Setup budget nhưng làm trầm trọng Hold violations; Negative Skew thì ngược lại.

![[Pasted image 20260521160507.png]]
## Computed from
Clock Skew phát sinh từ sự bất đối xứng vật lý trong Clock Tree sau CTS:

$$\delta_{skew} = \text{Latency}_{capture} - \text{Latency}_{launch}$$

Trong đó Latency của mỗi FF = Source Latency + Network Latency (toàn bộ delay từ clock source đến clock pin của FF đó). STA tool đo Skew bằng cách so sánh Clock Arrival Time tại từng cặp (Launch FF, Capture FF) liên quan đến cùng một timing path. Có 3 loại Skew thường gặp trong thực tế: **Local Skew** (giữa 2 FF cụ thể trên cùng một timing path — con số STA dùng để tính [[Slack]]), **Global Skew** (max − min Latency trên toàn bộ clock domain — metric đánh giá chất lượng CTS), **Useful Skew** (Skew được cố tình tạo ra để cải thiện timing trên critical paths).

## Constrains
- **[[SetupTime]]**: Positive Skew cộng thêm vào RAT_setup: $RAT = T_c + \delta_{skew} - t_{setup} - \delta_{margin}$ — nới rộng timing budget cho data path; đây là nguyên lý của kỹ thuật Useful Skew
- **[[HoldTime]]**: Positive Skew cộng thêm vào RAT_hold: $RAT_{hold} = t_{hold} + \delta_{skew}$ — thu hẹp Hold budget; Hold violations bùng phát sau CTS chính vì Skew thực tế xuất hiện
- **[[Slack]]**: Skew xuất hiện trong cả Setup Slack và Hold Slack equations với chiều tác động ngược nhau — cải thiện Setup đồng thời làm xấu Hold

## Requires
- [[ClockTreeSynthesis]] — Skew là hệ quả trực tiếp của Clock Tree structure; trước CTS, STA giả định ideal clock (Skew = 0); sau CTS, Skew thực tế được propagated vào timing analysis
- [[ClockLatency]] — Skew được tính từ hiệu số Latency giữa hai FF; không thể có Skew nếu không có Latency per-FF

## Used by
- [[STA]] — annotate Skew vào timing graph sau CTS; dùng để adjust Setup RAT và Hold RAT trên từng timing path; Local Skew per path-pair là con số STA thực sự dùng
- [[ClockTreeSynthesis]] — Global Skew là primary quality metric của CTS; CTS tool iterate để minimize Global Skew trong khi đáp ứng Slew và Latency constraints
- [[Signoff]] — Skew tại Signoff được compute từ propagated clock với full SPEF; là thành phần của Clock Uncertainty budget verification

## Key insight
[USER REVIEW — draft suggestion]: Clock Skew là khái niệm trung tâm phân biệt Pre-CTS timing (lý tưởng) và Post-CTS timing (thực tế). Người mới thường chỉ nhớ "Skew xấu" — nhưng thực tế Skew có thể được exploit có chủ đích: Useful Skew là kỹ thuật cố tình tạo ra positive Skew trên critical paths để cải thiện Setup Slack mà không cần thay đổi logic. Trade-off là Hold Slack xấu đi — nên Useful Skew chỉ được dùng khi Hold margin còn dư. Thực tế quan trọng: Hold violations sau CTS gần như luôn có nguyên nhân chính là Clock Skew, không phải logic quá nhanh.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Produced by: [[ClockTreeSynthesis]]
→ Component of: [[ClockUncertainty]] (pre-CTS modeling)
→ Affects: [[SetupTime]] · [[HoldTime]] · [[Slack]]
→ Related: [[ClockLatency]] · [[ClockJitter]] · [[ClockUncertainty]]
→ Cùng nhóm: [[ClockUncertainty]] · [[ClockLatency]] · [[ClockJitter]] · [[STA]] · [[SDC]]