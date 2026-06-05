---
tags: [concept, sta-timing, process-variation]
group: STA — Timing
defined_in: STA tool — CPPR adjustment trong OCV analysis mode
used_by: [STA, PreCTSOptimization, Signoff]
requires: [OCV, ClockSkew, STA]
chain: Chain_STA_Basics
---
# CPPR

## Definition
CPPR (Common Path Pessimism Removal, hoặc Clock Reconvergence Pessimism Removal) là kỹ thuật loại bỏ bi quan không thực tế phát sinh từ [[OCV]] mode cho phần clock path chung (common path) giữa launch và capture FF. Trong OCV, common clock path bị tính hai lần với hai delay khác nhau (max cho launch side, min cho capture side) — điều này không phản ánh thực tế vật lý vì một đoạn clock path cụ thể chỉ có một delay tại bất kỳ thời điểm nào.

## Physical Context

Trong clock tree, launch FF và capture FF thường chia sẻ một đoạn clock path từ root đến điểm phân nhánh:

CLK → [c1] → [c2] → n1  ← điểm phân nhánh
├── [c3] → [c4] → Launch FF
└── [c5] → Capture FF

**Common path** = đoạn CLK → n1 (qua c1, c2).
**Divergent path** = đoạn từ n1 về mỗi FF riêng.

## Mechanism of Pessimism in OCV

Trong OCV mode, Setup analysis:
- Launch clock path dùng **max delay**: common path = max + divergent path launch = max
- Capture clock path dùng **min delay**: common path = min + divergent path capture = min

Ví dụ minh họa (từ L7 p.57), mỗi cell có max delay = 1 ns, min delay = 0.8 ns, common path = c1 + c2:

| Path | Delay | Tính toán |
|---|---|---|
| Launch (max) | 1 + 1 + 1 = **3 ns** | common (max) + divergent |
| Capture (min) | 0.8 + 0.8 + 0.8 = **2.4 ns** | common (min) + divergent |

**Clock skew không có CPPR** = 3 − 2.4 = **0.6 ns**

Tuy nhiên, c1 và c2 là các cell vật lý xác định — chúng không thể đồng thời có delay 1 ns (cho launch) và 0.8 ns (cho capture). Bi quan phát sinh từ đoạn common path bị tính hai lần.

## CPPR Adjustment Calculation

$$\text{CPPR adjustment} = \text{max delay (common path)} - \text{min delay (common path)}$$

$$= (1 + 1) - (0.8 + 0.8) = 2.0 - 1.6 = 0.4 \text{ ns}$$

**Clock skew với CPPR:**
$$\text{skew} = 3 - 2.4 - 0.4 = \textbf{0.2 ns}$$

0.2 ns chỉ phản ánh variation trên divergent path — phần thực sự uncertain — thay vì 0.6 ns bao gồm cả bi quan không thực tế từ common path.

## Impact on Slack

**Setup:** CPPR adjustment được cộng vào RAT (trả lại budget bị mất do pessimism):
$$\text{Setup Slack}_{CPPR} = \text{Setup Slack}_{no CPPR} + \text{CPPR adjustment}$$

**Hold:** CPPR adjustment được trừ khỏi RAT (giảm bớt bi quan theo chiều ngược):
$$\text{Hold Slack}_{CPPR} = \text{Hold Slack}_{no CPPR} - \text{CPPR adjustment}$$

Vì vậy CPPR cải thiện Setup margin và có thể làm Hold margin xấu đi một chút — đây là trade-off cần lưu ý.

## CPPR trong Timing Report
Trong timing report, CPPR adjustment xuất hiện như một dòng riêng:

Setup:          -0.152
Cppr Adjust:    +0.000   ← CPPR = 0 khi không có common path hoặc CPPR chưa được bật
Required Time:   3.848

Khi CPPR được bật và có common path, giá trị `Cppr Adjust` sẽ dương (trả lại timing budget cho Setup).

## Command
```tcl
set_db timing_analysis_cppr both
```
`-both` áp CPPR cho cả Setup và Hold. Các option: `-setup`, `-hold`, `-both`, hoặc `none` (tắt CPPR).

## Requires
- [[OCV]] — CPPR chỉ có nghĩa trong OCV mode; Single và BC-WC không có phân biệt max/min per-path nên không có common path pessimism
- [[ClockSkew]] — CPPR trực tiếp ảnh hưởng đến effective clock skew trong timing analysis
- [[STA]]

## Used by
- [[STA]] — CPPR adjustment được apply trong timing computation; giá trị xuất hiện trong timing report như `Cppr Adjust`
- [[PreCTSOptimization]] — phải enable cùng với OCV trước khi chạy `opt_design -pre_cts`
- [[Signoff]] — CPPR thường được bật ở production signoff để có kết quả chính xác hơn

## Related
→ Chain: [[Chain_STA_Basics]]
→ Closely related: [[OCV]] — CPPR giải quyết pessimism do OCV gây ra
→ Related: [[ClockSkew]] · [[STA]] · [[PreCTSOptimization]] · [[ClockTreeSynthesis]]
→ Cùng nhóm: [[OCV]] · [[ClockSkew]] · [[ClockUncertainty]] · [[STA]]