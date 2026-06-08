---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — quality metric và design constraint
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview, STA, Signoff]
requires: [ClockTreeSynthesis, Slew, OCV]
chain: Chain_PnR_Flow
---
# ClockDutyCycle

## Definition
Clock Duty Cycle là tỷ lệ phần trăm của một clock period mà trong đó clock signal duy trì trạng thái active (high):

$$\text{Duty Cycle} = \frac{T_{on}}{T_{period}} \times 100\%$$

Ideal duty cycle = 50%, tức clock signal dành bằng nhau cho trạng thái high ($T_{on}$) và low ($T_{off}$) trong mỗi chu kỳ. **Duty cycle degradation** là hiện tượng tỷ lệ này lệch khỏi 50% do các yếu tố vật lý tích lũy dọc clock path.

## Why Duty Cycle Matters
Duty cycle degradation ảnh hưởng trực tiếp đến:
- **Half-cycle timing paths**: paths launch trên một sườn và capture trên sườn đối diện chỉ có nửa chu kỳ để data propagate. Nếu half period bị thu ngắn do duty cycle lệch, timing margin giảm theo — xem [[HalfCyclePath]].
- **Minimum pulse width checks**: clock pulse phải đủ rộng để sequential cells hoạt động đúng. Duty cycle xấu làm thu hẹp pulse width — xem [[MinimumPulseWidth]].
- **Latch-based designs**: latch dùng toàn bộ half-cycle làm transparency window; duty cycle không đều làm window bất đối xứng.
- **Clock reliability**: clock signal bị distort trở nên không predictable, ảnh hưởng toàn bộ clock quality.

## Causes of Duty Cycle Degradation
- **Rise/fall transition mismatch**: tốc độ chuyển lên và chuyển xuống không bằng nhau dọc clock path → $T_{on} \neq T_{off}$.
- **Unequal PMOS/NMOS drive strength**: trong clock buffer, nếu PMOS (pull-up) và NMOS (pull-down) có drive strength không tương đương, output transition bất đối xứng.
- **OCV**: biến thiên process trong-chip làm transistors trên clock path có đặc tính không đồng đều → bất đối xứng rise/fall delay tích lũy.
- **Crosstalk và noise**: coupling từ aggressor nets có thể distort clock waveform.
- **Excessive clock buffering**: nhiều buffer stages tích lũy sai số rise/fall theo từng stage.
- **PVT variation**: Process, Voltage, Temperature variation làm PMOS/NMOS mất cân bằng.

## Clock Buffer vs Regular Buffer
Clock Buffer được thiết kế đặc biệt để minimize duty cycle degradation thông qua:
- Matched PMOS/NMOS drive strength → balanced rise/fall transition → stable duty cycle (~50%).
- Symmetric delay cho cả hai sườn clock.

Regular Buffer (general-purpose) không được tối ưu cho rise/fall balance → unequal $T_{on}$/$T_{off}$ → duty cycle distortion. Đây là lý do CTS phải dùng clock-specific buffers, không dùng regular buffers — xem [[NDR]] để biết cách áp dụng routing rules đặc biệt cho clock nets.

## Requires
- [[ClockTreeSynthesis]]
- [[Slew]] — rise/fall transition mismatch là nguyên nhân chính; Slew degradation dọc clock tree tích lũy thành duty cycle error
- [[OCV]] — on-chip variation ảnh hưởng PMOS/NMOS balance trong clock cells

## Used by
- [[ClockTreeSynthesis]] — CTS minimize duty cycle degradation bằng cách chọn matched clock buffers
- [[CTSOptimization]] — cell selection cho clock cells phải cân bằng rise/fall delay
- [[CTSQualityReview]] — duty cycle là quality metric cần review sau CTS
- [[STA]] — half-cycle path timing check phụ thuộc actual duty cycle
- [[Signoff]] — clock duty cycle verification là một phần của clock quality signoff

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Formula: $\text{Duty Cycle} = T_{on}/T_{period} \times 100\%$; ideal = 50%
→ Primary cause: rise/fall transition mismatch tích lũy dọc clock path
→ Closely related: [[MinimumPulseWidth]] · [[HalfCyclePath]] · [[Slew]] · [[ClockTreeSynthesis]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockLatency]] · [[ClockUncertainty]] · [[ClockTreeSynthesis]]