---
tags: [concept, sta-timing, process-variation]
group: STA — Timing
defined_in: STA tool — timing analysis mode setting
used_by: [STA, PreCTSOptimization, MMMC, Signoff]
requires: [STA, LIB, SPEF]
chain: Chain_STA_Basics
---
# OCV

## Definition
OCV (On-Chip Variation) là timing analysis mode trong STA phản ánh thực tế rằng các transistor được chế tạo trên cùng một wafer nhưng ở vị trí khác nhau có thể có đặc tính điện khác nhau do biến thiên trong quá trình fabrication (chiều dài kênh, chiều rộng, độ dày oxide gate). OCV cho phép tool dùng max delay cho một path và min delay cho path kia — thay vì giả định cùng một delay cho toàn bộ chip.

## Sources of Process Variation
Process variation xảy ra ở hai mức:
- **Chip-to-chip**: giữa các dies/wafers/lots — đây là variation quen thuộc được capture bởi PVT corners trong [[LIB]]
- **On-chip**: trong cùng một die — đây là vấn đề OCV giải quyết

On-chip variation trở thành vấn đề đáng kể từ ~130nm trở xuống và ngày càng nghiêm trọng hơn ở các process nodes nhỏ hơn, do lithography variation, chemical mechanical planarization (CMP) non-uniformity, và implantation variation.

## Three Timing Analysis Modes

**1. Single**
Tool dùng một bộ delay duy nhất từ một library set cho cả Setup và Hold. Không phân biệt path-to-path variation. Ít bi quan nhất, chỉ phù hợp cho estimation thô.

**2. Best_Case_Worst_Case (BC-WC)**
- Setup check: max delay cho tất cả paths
- Hold check: min delay cho tất cả paths

BC-WC bi quan hơn Single nhưng vẫn áp cùng delay mode cho toàn bộ timing path — không phân biệt launch vs capture.

**3. OCV (On-Chip Variation)**
- Setup check: **max delay** cho launch path, **min delay** cho capture path
- Hold check: **min delay** cho launch path, **max delay** cho capture path

OCV phản ánh worst-case thực tế: launch FF và capture FF nằm ở các vùng silicon khác nhau nên có thể bị ảnh hưởng bởi variation theo chiều ngược nhau. Đây là mode được dùng rộng rãi nhất trong PD flow từ 130nm trở xuống.

$$\text{Setup AT} = t_{cq,max} + \sum t_{stage,max} \quad \text{(launch path: max delay)}$$
$$\text{Setup RAT} = T_c + t_{capture\_clock,min} - t_{setup} - \delta_{margin} \quad \text{(capture clock: min delay)}$$

## Relation to CPPR
Trong OCV mode, common clock path (đoạn clock tree chia sẻ giữa launch và capture) bị tính hai lần với hai delay khác nhau — tạo ra bi quan không thực tế. [[CPPR]] giải quyết vấn đề này bằng cách loại bỏ phần pessimism dư thừa trên common path.

## Command
```tcl
set_db timing_analysis_type ocv
```

## OCV vs AOCV vs POCV
OCV flat dùng một hệ số derating đồng đều cho mọi path. Các refinements:
- **AOCV (Advanced OCV)**: hệ số derating phụ thuộc độ dài path — path dài hơn có deviation nhỏ hơn do statistical averaging
- **POCV (Parametric OCV)**: dùng statistical distribution thay vì worst-case bounds — chính xác nhất cho signoff ở advanced nodes

## Requires
- [[STA]]
- [[LIB]]
- [[SPEF]]

## Used by
- [[STA]] — mode phân tích quyết định cách tool annotate max/min delay cho launch/capture paths
- [[PreCTSOptimization]] — phải enable trước khi chạy `opt_design -pre_cts`
- [[MMMC]] — tương tác với Delay Corner selection (dc_max cho Setup, dc_min cho Hold)
- [[Signoff]] — Signoff STA luôn chạy với OCV hoặc AOCV/POCV

## Related
→ Chain: [[Chain_STA_Basics]]
→ Closely related: [[CPPR]] — giải quyết pessimism do OCV gây ra trên common clock path
→ Related: [[ClockSkew]] · [[STA]] · [[MMMC]] · [[PreCTSOptimization]]
→ Evolution: OCV → AOCV → POCV (độ chính xác tăng dần)
→ Cùng nhóm: [[CPPR]] · [[ClockSkew]] · [[ClockUncertainty]] · [[STA]]