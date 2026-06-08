---
tags: [concept, sta-timing, clock]
group: STA — Clock
defined_in: SDC — set_clock_uncertainty command; consumed bởi STA tool
used_by: [STA, LogicSynthesis, Signoff]
requires: [SDC, ClockJitter, ClockSkew]
chain: Chain_STA_Basics
---
# ClockUncertainty

## Definition
Clock Uncertainty là tham số SDC dùng để model tổng sự không chắc chắn về thời điểm clock edge thực sự đến tại Flip-flop. Clock uncertainty tổng hợp bốn nguồn không chắc chắn: **Jitter** (biến thiên chu kỳ từ PLL), **Skew variation** (chênh lệch arrival time giữa các FF), **PVT variation** (biến thiên Process, Voltage, Temperature làm timing không đồng nhất trên-chip), và **additional setup/hold timing margin** để dự phòng. Clock Uncertainty được subtract ra khỏi timing budget như một safety margin, buộc STA tool phải optimize design với biên độ dự phòng đủ lớn để chip hoạt động an toàn dù có nhiễu và biến thiên vật lý. Quan trọng: Clock Uncertainty cho Setup và Hold được set khác nhau vì bản chất vật lý của Jitter khác nhau trong hai bài toán.

## Computed from
Clock Uncertainty được kỹ sư thiết lập trong SDC dựa trên specs từ PLL datasheet và CTS quality targets:

**For Setup check:**
$$\delta_{uncertainty,setup} = \text{Jitter} + \text{Worst Clock Skew (budget)}$$

Jitter được cộng vào vì Setup check đánh giá hai sườn clock khác nhau (sườn n và sườn n+1) — PLL có thể bị nhiễu khiến sườn n+1 đến sớm hơn dự kiến, thu hẹp timing window.

**For Hold check:**
$$\delta_{uncertainty,hold} = 0 + \text{Best Clock Skew (budget)}$$

Jitter = 0 vì Hold check đánh giá trên cùng một sườn clock — một sườn clock không thể Jitter đối với chính nó. Chỉ có Skew còn lại.

Trong STA, Uncertainty được subtract từ RAT:
- Setup: $RAT_{setup} = T_c + \delta_{skew} - t_{setup} - \delta_{uncertainty,setup}$
- Hold: $RAT_{hold} = t_{hold} + \delta_{skew} + \delta_{uncertainty,hold}$

Trước CTS (`set_ideal_network`): Uncertainty lớn hơn vì phải estimate cả Skew chưa biết. Sau CTS (`set_propagated_clock`): Skew được compute thực tế từ Clock Tree — Uncertainty chỉ còn Jitter + small margin.

## Constrains
- **[[Slack]]**: Uncertainty làm giảm RAT → thu hẹp Slack; Uncertainty càng lớn, design càng khó close timing — nhưng nếu set quá nhỏ, chip có thể fail silicon test
- **[[SetupTime]]**: Setup Uncertainty bao gồm Jitter → buộc synthesis và PD phải optimize tighter; sau CTS có thể reduce Setup Uncertainty khi Skew thực tế đã được tính
- **[[HoldTime]]**: Hold Uncertainty = 0 Jitter → nhỏ hơn Setup Uncertainty; nhưng Hold Skew component vẫn làm Hold violations khó fix hơn

## Requires
- [[SDC]] — `set_clock_uncertainty -setup value` và `set_clock_uncertainty -hold value` là SDC commands tạo ra Uncertainty; giá trị do kỹ sư thiết lập dựa trên PLL specs và CTS targets
- [[ClockJitter]] — thành phần vật lý đầu tiên của Setup Uncertainty; giá trị từ PLL datasheet hoặc characterization
- [[ClockSkew]] — thành phần vật lý thứ hai; pre-CTS là budget estimate, post-CTS là actual measured Skew

## Used by
- [[STA]] — đọc Uncertainty từ SDC để adjust RAT computation; Setup và Hold paths dùng Uncertainty values khác nhau; `check_timing` verify Uncertainty đã được set cho tất cả clocks
- [[LogicSynthesis]] — synthesis STA dùng pre-CTS Uncertainty (lớn hơn) để account cho Skew chưa biết; đây là một lý do synthesis timing thường pessimistic hơn post-route timing
- [[Signoff]] — Signoff verify rằng chip pass timing với Uncertainty đã được set đúng theo PLL specs và process variation; nếu Uncertainty set quá nhỏ, silicon test fail rate cao

## Key insight
[USER REVIEW — draft suggestion]: Clock Uncertainty là nơi kỹ sư "dịch" những bất định vật lý (Jitter từ PLL, Skew từ Clock Tree) thành ngôn ngữ toán học mà STA tool hiểu được. Điểm quan trọng nhất: Jitter = 0 cho Hold check không phải là approximation hay simplification — đây là sự thật vật lý chính xác, vì Hold check xét trên cùng một clock edge, và một edge không thể Jitter so với chính nó. Hiểu điều này giải thích tại sao Hold violations thường dễ predict hơn Setup violations (ít biến số hơn), nhưng lại nguy hiểm hơn vì không có workaround sau silicon.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Components: [[ClockJitter]] + [[ClockSkew]]
→ Set by: [[SDC]] — set_clock_uncertainty
→ Affects: [[SetupTime]] · [[HoldTime]] · [[Slack]]
→ Related: [[ClockSkew]] · [[ClockLatency]] · [[ClockJitter]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockLatency]] · [[ClockJitter]] · [[STA]] · [[SDC]]