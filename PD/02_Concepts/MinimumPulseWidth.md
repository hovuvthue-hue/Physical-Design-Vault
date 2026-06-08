---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — timing check type và quality criterion
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview, PostCTSOptimization, STA, Signoff]
requires: [ClockTreeSynthesis, ClockDutyCycle, Slew, LIB]
chain: Chain_PnR_Flow
---
# MinimumPulseWidth

## Definition
Minimum Pulse Width là khoảng thời gian tối thiểu mà clock signal phải duy trì ở trạng thái high (high time) hoặc low (low time) để sequential elements trong design hoạt động đúng. Yêu cầu này được Foundry characterize và lưu trong Standard Cell Library cho từng sequential cell type tại từng PVT corner.

## Minimum Pulse Width Requirement
Tất cả sequential elements (Flip-flop, Latch) đều yêu cầu minimum clock pulse width. Clock pulse phải duy trì trên ngưỡng minimum do library quy định; nếu ngắn hơn, cell có thể không chốt dữ liệu đúng cách.

Giá trị yêu cầu phụ thuộc vào: technology node, standard cell library, operating voltage, và process conditions.

Violating minimum pulse width có thể gây:
- **Incorrect data capture**: Flip-flop không chốt đúng giá trị → lỗi logic
- **Metastability**: ngõ ra Q rơi vào trạng thái không xác định
- **Functional failures**: lỗi chức năng không thể predict được

## Minimum Pulse Width Check
Pulse width check xác nhận clock high time và clock low time đều thỏa mãn ngưỡng minimum:

$$\text{Actual Pulse Width} \geq \text{Required Pulse Width (from LIB)}$$

$$\text{Pulse Width Slack} = \text{Actual Pulse Width} - \text{Required Pulse Width}$$

Slack âm → Pulse Width Violation. Check này chạy song song với Setup/Hold check trong STA, là một timing check type riêng biệt.

## Causes of Minimum Pulse Width Violations
- **Duty cycle degradation**: bất đối xứng rise/fall thu hẹp một nửa chu kỳ — xem [[ClockDutyCycle]]
- **Excessive clock slew**: Slew thoải làm clock waveform méo → effective pulse width giảm
- **Skew imbalance**: clock skew không đồng đều làm một số sinks nhận pulse ngắn hơn
- **Clock signal distortion**: noise hoặc crosstalk distort clock waveform

## Fixes
**1. Dùng balanced clock buffers và matched inverter structures trong CTS:**
Matched inverter pairs (cặp 2 inverters liên tiếp) có tính chất tự cân bằng rise/fall delay, minimize duty cycle distortion qua clock tree.

**2. Chèn thêm buffers hoặc inverters tại vị trí cụ thể trên clock tree:**
Bổ sung clock cells tại các nhánh có pulse width violation để điều chỉnh lại duty cycle cục bộ.

## When Pulse Width Checks Are Important
Pulse width checks phải được thực hiện tại:
- **CTS** — ngay trong quá trình build clock tree để tránh tạo ra violations
- **Post-CTS optimization** — cleanup violations phát sinh sau khi clock tree hoàn thành
- **Signoff timing analysis** — tất cả clocks phải pass minimum pulse width requirements trước tape-out

## Requires
- [[ClockTreeSynthesis]] — violations thường phát sinh hoặc được phát hiện sau CTS
- [[ClockDutyCycle]] — nguyên nhân chính của minimum pulse width violations
- [[Slew]] — excessive clock slew làm méo waveform, thu hẹp effective pulse width
- [[LIB]] — required pulse width được characterize và lưu trong LIB cho từng sequential cell

## Used by
- [[ClockTreeSynthesis]] — CTS phải đảm bảo pulse width requirement được đáp ứng tại tất cả sinks
- [[CTSOptimization]] — pulse width là một optimization target
- [[CTSQualityReview]] — pulse width Slack là quality metric sau CTS
- [[PostCTSOptimization]] — cleanup pulse width violations còn lại sau CTS
- [[STA]] — pulse width là một timing check type (song song với setup/hold)
- [[Signoff]] — tất cả clocks phải pass minimum pulse width trước tape-out

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Check: Actual Pulse Width ≥ Required Pulse Width (from LIB)
→ Violations caused by: [[ClockDutyCycle]] degradation · excessive [[Slew]] · skew imbalance
→ Fixed by: matched clock buffers · matched inverter pairs · targeted buffer insertion
→ Closely related: [[ClockDutyCycle]] · [[ClockTreeSynthesis]] · [[CTSOptimization]] · [[Slew]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockLatency]] · [[ClockUncertainty]] · [[ClockTreeSynthesis]]