---
tags: [concept, pnr-flow, cts, optimization]
group: PnR Flow
defined_in: ClockTreeSynthesis
used_by: [ClockTreeSynthesis, STA, Signoff]
requires: [ClockTreeSynthesis, ClockSkew, ClockLatency, Slew, Slack]
chain: Chain_PnR_Flow
---
# CTSOptimization

## Definition
**CTSOptimization** là tập các kỹ thuật tối ưu được áp dụng trong hoặc xung quanh bước [[ClockTreeSynthesis]] để cải thiện chất lượng clock tree ở các trục chính: [[ClockSkew]], [[ClockLatency]], [[Slew]]/transition, và rủi ro timing thể hiện qua [[Slack]].

Mục tiêu không phải chỉ “giảm skew bằng mọi giá”, mà là cân bằng giữa timing, điện năng, tài nguyên routing và độ ổn định của flow.

## Why CTS optimization is needed
Sau CTS, clock không còn là ideal clock mà trở thành propagated clock với network vật lý thực. Lúc này:
- độ lệch arrival time giữa các sinks trở nên rõ ràng hơn,
- transition/cap/fanout violations trên clock path có thể lộ ra,
- một số setup/hold risk có thể tăng rõ trong post-CTS [[STA]].

Vì vậy cần một lớp tối ưu chuyên biệt để đưa clock tree về trạng thái cân bằng hơn trước các bước downstream như [[Routing]] và [[Signoff]].

## Main optimization techniques
### 1) Cell Relocation
Di chuyển vị trí vật lý của clock cells (buffer/inverter) để phân bố lại khoảng cách đến sink clusters, từ đó cải thiện skew và transition theo hướng cân bằng hơn.

### 2) Level Adjustment
Điều chỉnh “mức” của sink/nhánh trong clock tree hierarchy để thay đổi arrival time tương đối giữa các sink mà không nhất thiết phải tái cấu trúc toàn cây.

### 3) Reconfiguration
Tái tổ chức connectivity hoặc phân cụm sink giữa các nhánh clock tree để cải thiện load balance, skew, hoặc electrical quality tổng thể.

### 4) Dummy Load Insertion
Bổ sung tải giả có kiểm soát ở một số nhánh clock để cân bằng delay giữa các nhánh nhanh/chậm. Kỹ thuật này có thể giúp cân bằng timing nhưng thường tăng switching power và tải tổng.

### 5) Load Splitting
Tách tải lớn (fanout/capacitance lớn) thành nhiều nhánh nhỏ hơn bằng cách thêm stages hoặc tái phân phối sink, nhằm cải thiện slew/DRV trên từng nhánh.

### 6) Useful Skew (as controlled skew)
Thay vì ép skew về gần 0 tuyệt đối, có thể chủ động điều chỉnh skew có kiểm soát trên một số timing paths để hỗ trợ closure.

Useful skew có thể được cân nhắc khi một hard setup path cần thêm effective timing budget, nhưng đây là trade-off CTS/STA có kiểm soát chứ không phải functional timing exception. Useful skew có thể cải thiện setup margin trên các path được chọn đồng thời làm giảm hold margin trên các path liên quan; vì vậy phải đánh giá setup và hold cùng nhau qua [[HoldTime]], [[Slack]], và [[STA]]. Mức áp dụng thực tế phụ thuộc tool/flow và policy timing-closure. [Needs verification]

### 7) Buffering / Cell Sizing / VT-Swapping trong CTS context
- **Buffering**: thêm/tối ưu clock buffers để tăng drive, cải thiện transition và phân phối tải.
- **Cell Sizing**: thay đổi drive strength clock cells để cân bằng delay, skew, slew.
- **VT-Swapping**: thay đổi Vt flavor để trade-off delay và leakage theo mục tiêu timing/power.

Mức độ cho phép và hiệu quả của từng kỹ thuật phụ thuộc library, PDK, tool, và policy flow cụ thể. [Needs verification]

## Trade-offs and risks
- Cải thiện skew thường đi kèm tăng area/power do thêm cell hoặc tăng size.
- Cải thiện setup qua skew tuning có thể làm hold margin xấu hơn ở path khác.
- Sửa transition/DRV bằng thêm buffers có thể tăng capacitance, congestion, và complexity cho downstream [[Routing]].
- Một số kỹ thuật có thể bị giới hạn bởi CTS spec, clock architecture, library availability, hoặc flow policy. [Needs verification]

## Requires
- [[ClockTreeSynthesis]]
- [[ClockSkew]]
- [[ClockLatency]]
- [[Slew]]
- [[Slack]]
- [[HoldTime]]

## Used by
- [[ClockTreeSynthesis]]
- [[STA]] (post-CTS evaluation và timing-risk balancing)
- [[Signoff]] (đánh giá tác động timing/power/quality sau các vòng tối ưu)

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Closely related: [[ClockTreeSynthesis]] · [[ClockSkew]] · [[ClockLatency]] · [[Slew]] · [[HoldTime]] · [[Slack]] · [[STA]] · [[Signoff]]
