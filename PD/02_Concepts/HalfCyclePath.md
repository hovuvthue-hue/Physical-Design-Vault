---
tags: [concept, sta-timing, clock]
group: STA — Timing
defined_in: STA tool — timing path classification theo clock edge relationship
used_by: [STA, ClockTreeSynthesis, ClockDutyCycle, Signoff]
requires: [SetupTime, HoldTime, ClockDutyCycle]
chain: Chain_STA_Basics
---
# HalfCyclePath

## Definition
Half-Cycle Path là timing path trong đó data được launch trên một sườn clock và capture trên sườn đối diện (positive-edge → negative-edge hoặc negative-edge → positive-edge). Kết quả là data chỉ có **nửa clock period** để propagate thay vì một full cycle như Single-Cycle Path.

Phân biệt với **Single-Cycle Path**: data launch và capture cùng edge type (positive→positive hoặc negative→negative) — full clock period khả dụng cho data propagation.

## Setup Timing for Half-Cycle Path
Với half-cycle path, timing budget cho data propagation bị thu hẹp xuống còn nửa chu kỳ:

$$\text{Setup constraint: } T_{data} \leq \frac{T_{period}}{2} - t_{setup}$$

Đây là constraint nghiêm hơn đáng kể so với single-cycle path (budget = $T_{period}$), nên half-cycle paths thường là critical paths trong design có cả positive-edge và negative-edge triggered elements.

## Hold Timing for Half-Cycle Path
Hold constraint cho half-cycle path lại được **nới lỏng**:

$$\text{Hold constraint: } T_{data,min} \geq t_{hold} - \frac{T_{period}}{2}$$

Vì capture edge cách launch edge nửa chu kỳ, data cũ đã có thêm nửa chu kỳ "dự phòng" trước khi capture edge xuất hiện — nên khó có Hold violation hơn.

## Impact of Duty Cycle Degradation on Half-Cycle Paths
Nếu clock duty cycle không phải 50%, hai nửa chu kỳ có độ dài không bằng nhau ($T_{on} \neq T_{off}$). Half-cycle path setup budget trở thành $T_{on}$ hoặc $T_{off}$ tùy chiều đi của data — một trong hai nửa ngắn hơn nominal $T_{period}/2$. Đây là lý do [[ClockDutyCycle]] degradation ảnh hưởng trực tiếp đến half-cycle path timing closure.

## Common Sources of Half-Cycle Paths
- Design dùng cả positive-edge và negative-edge Flip-flops (ví dụ: DDR interface)
- Latch-based pipelines trong một số high-performance designs
- Mixed flip-flop/latch topologies

## Requires
- [[SetupTime]] — half-cycle path tightens setup constraint xuống nửa chu kỳ
- [[HoldTime]] — half-cycle path relaxes hold constraint
- [[ClockDutyCycle]] — duty cycle degradation ảnh hưởng trực tiếp đến timing budget của half-cycle paths

## Used by
- [[STA]] — STA nhận diện half-cycle paths và áp dụng timing constraints phù hợp; setup check budget = T_period/2 thay vì T_period
- [[ClockTreeSynthesis]] — CTS phải đảm bảo duty cycle tốt để không làm xấu timing margin của half-cycle paths
- [[ClockDutyCycle]] — duty cycle degradation tác động trực tiếp đến half-cycle path setup timing
- [[Signoff]] — half-cycle path violations phải được clean tại signoff

## Related
→ Chain: [[Chain_STA_Basics]]
→ Contrasted with: Single-Cycle Path (same edge type, full T_period budget)
→ Setup: tightened to T_period/2; Hold: relaxed by T_period/2
→ Sensitive to: [[ClockDutyCycle]] degradation
→ Closely related: [[SetupTime]] · [[HoldTime]] · [[ClockDutyCycle]] · [[STA]]
→ Cùng nhóm: [[SetupTime]] · [[HoldTime]] · [[Slack]] · [[ClockSkew]]