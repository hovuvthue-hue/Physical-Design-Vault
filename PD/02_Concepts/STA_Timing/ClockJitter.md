---
tags: [concept, sta-timing, clock]
group: STA — Clock
defined_in: Đặc tính vật lý của PLL/oscillator — measured bởi circuit characterization; modeled trong SDC qua set_clock_uncertainty
used_by: [STA, SDC, Signoff]
requires: [SDC]
chain: Chain_STA_Basics
---
# ClockJitter

## Definition
Clock Jitter là sự thay đổi ngẫu nhiên về thời điểm xuất hiện của clock edges từ chu kỳ này sang chu kỳ khác, so với vị trí lý tưởng của chúng. Nguyên nhân nội tại của Jitter nằm trong bộ tạo dao động (PLL hoặc crystal oscillator): nhiễu nguồn điện, biến thiên nhiệt độ, và nhiễu nền của transistor trong PLL khiến các clock edges bị dịch chuyển ngẫu nhiên trái/phải so với vị trí lý tưởng. Jitter là đặc tính của clock source — không phải của Clock Tree hay routing.

## Computed from
Jitter được characterize bởi nhà sản xuất PLL và cung cấp trong datasheet dưới dạng:
- **Period Jitter**: độ lệch của clock period so với nominal period
- **Cycle-to-Cycle Jitter**: độ lệch giữa hai chu kỳ liên tiếp
- **Long-term Jitter / Phase Noise**: tích lũy qua nhiều chu kỳ

Trong STA, Jitter ảnh hưởng đến timing analysis theo cơ chế sau:

**Với Setup check** (đánh giá sườn clock thứ n và n+1):
Jitter làm sườn n+1 có thể đến sớm hơn dự kiến → thu hẹp timing window → phải subtract Jitter khỏi timing budget:
$$\delta_{uncertainty,setup} = \text{Jitter} + \text{Clock Skew budget}$$

**Với Hold check** (đánh giá trên cùng một sườn clock):
Một clock edge không thể Jitter so với chính nó → Jitter contribution = 0:
$$\delta_{uncertainty,hold} = 0 + \text{Clock Skew budget}$$

Đây là lý do Setup Uncertainty luôn lớn hơn Hold Uncertainty bằng đúng giá trị Jitter.

## Constrains
- **[[ClockUncertainty]]**: Jitter là thành phần tạo nên ClockUncertainty cho Setup check; giá trị Jitter từ PLL datasheet là input để kỹ sư set `set_clock_uncertainty -setup` trong SDC
- **[[SetupTime]]**: Jitter thu hẹp effective timing budget cho Setup — chip chạy ở tần số cao hơn với PLL có Jitter lớn hơn sẽ khó close timing hơn
- **[[HoldTime]]**: Jitter không ảnh hưởng đến Hold check — đây là sự thật vật lý chính xác, không phải approximation

## Requires
- [[SDC]] — Jitter được model vào STA thông qua `set_clock_uncertainty -setup` trong SDC; STA tool không đo Jitter trực tiếp mà nhận giá trị từ SDC do kỹ sư thiết lập dựa trên PLL specs

## Used by
- [[STA]] — Jitter (thông qua Clock Uncertainty) được subtract từ RAT_setup trong timing calculation; làm cho Setup timing analysis pessimistic hơn so với Hold timing analysis
- [[SDC]] — `set_clock_uncertainty -setup value` where value = Jitter + Skew_budget; kỹ sư phải biết Jitter spec của PLL để set đúng giá trị này
- [[Signoff]] — Signoff verify rằng chip pass timing với Jitter budget đã được fold vào SDC; nếu PLL thực tế có Jitter lớn hơn estimate trong SDC, chip có thể fail timing test dù STA báo pass

## Key insight
[USER REVIEW — draft suggestion]: Jitter = 0 với Hold check là một trong những insights quan trọng nhất để hiểu sự bất đối xứng giữa Setup và Hold analysis. Lý do đơn giản nhưng sâu sắc: Jitter là biến thiên giữa các chu kỳ khác nhau — khi xét hai sự kiện trên cùng một clock edge, không có "giữa các chu kỳ" để nói đến. Hệ quả thực tế: Hold violations thường ổn định và reproducible (ít biến số hơn), trong khi Setup violations đôi khi chỉ xuất hiện ở điều kiện nhiệt độ cao hoặc khi PLL bị nhiễu nặng. Một con chip fail timing intermittently (thỉnh thoảng fail, không phải luôn luôn) thường là dấu hiệu của Jitter-induced Setup violation.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Source: PLL / crystal oscillator — đặc tính vật lý của clock generator
→ Component of: [[ClockUncertainty]] (Setup component only)
→ Affects: [[SetupTime]] only — không ảnh hưởng [[HoldTime]]
→ Modeled via: [[SDC]] — set_clock_uncertainty -setup
→ Related: [[ClockSkew]] · [[ClockUncertainty]] · [[ClockLatency]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockUncertainty]] · [[ClockLatency]] · [[STA]] · [[SDC]]