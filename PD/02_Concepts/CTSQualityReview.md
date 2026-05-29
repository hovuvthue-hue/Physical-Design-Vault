---
tags: [concept, pnr-flow, cts, signoff]
group: PnR Flow
defined_in: ClockTreeSynthesis
used_by: [Routing, Signoff, STA]
requires: [ClockTreeSynthesis, CTSFlow, ClockSkew, ClockLatency, Slew, PowerAnalysis, CongestionAnalysis]
chain: Chain_PnR_Flow
---
# CTSQualityReview

## Definition
**CTSQualityReview** là hoạt động đánh giá chất lượng đầu ra của [[ClockTreeSynthesis]] để quyết định liệu design đã đủ ổn định để đi tiếp xuống [[Routing]] và readiness cho [[Signoff]] hay chưa.

Mục tiêu của card này là mô tả khung đánh giá ở mức concept, không phải checklist cứng theo một tool cụ thể.

## What is reviewed after CTS
Sau CTS, review thường nhìn vào các nhóm output chính:
- netlist/design database đã được cập nhật với clock buffer/inverter,
- dữ liệu vật lý clock routing ở mức implementation database,
- thông tin timing và clock quality của clock network,
- ước lượng tác động lên power và routability downstream.

Tên report, định dạng database, và mức chi tiết theo từng tool phụ thuộc flow triển khai. [Needs verification]

## Timing and electrical checks
Nhóm kiểm tra timing/electrical thường bao gồm:
- trạng thái setup/hold slack sau propagated clock,
- DRV liên quan transition/[[Slew]], capacitance, fanout,
- mức ổn định QoR khi thay đổi corner/mode phân tích [[STA]].

Ngưỡng pass/fail cụ thể là flow-dependent, không nên xem như quy tắc phổ quát. [Needs verification]

## Placement and routability checks
CTS có thể chèn thêm nhiều clock cells, vì vậy quality review cũng cần xem:
- tính hợp lệ placement sau khi insert cell,
- ảnh hưởng lên mật độ và xu hướng nghẽn route,
- rủi ro tiêu tốn tài nguyên routing khi đi downstream.

Mức đánh giá congestion/routability cụ thể phụ thuộc tool metric và stage đo. [Needs verification]

## Power checks
Clock network thường là nguồn đóng góp dynamic power đáng kể, nên CTSQualityReview cần kiểm tra:
- tác động power do clock buffers/inverters được thêm,
- thay đổi switching load/capacitance của clock tree,
- xu hướng power khi cân bằng giữa timing và electrical cleanup.

Kết quả định lượng nên được đọc cùng [[PowerAnalysis]] trong đúng context stage/corner. [Needs verification]

## Clock quality checks
Các chỉ số clock quality cốt lõi gồm:
- [[ClockSkew]]
- [[ClockLatency]]
- [[Slew]] (clock transition)

Các chỉ số này cần được xem đồng thời thay vì tách rời, vì tối ưu một chiều có thể làm xấu chiều khác trong downstream closure.

## Debug aids
Một số flow dùng Clock Tree Debugger (CTD) như công cụ hỗ trợ quan sát clock tree topology và vùng bất thường.

CTD chỉ nên xem như debug aid phụ thuộc tool/UI, không phải định nghĩa cốt lõi của CTSQualityReview. [Needs verification]

## Limits / tool dependence
CTSQualityReview là khung khái niệm chung. Quy trình thực tế sẽ khác theo tool, library, PDK, signoff policy và target QoR của từng dự án. [Needs verification]

## Requires
- [[ClockTreeSynthesis]]
- [[CTSFlow]]
- [[ClockSkew]]
- [[ClockLatency]]
- [[Slew]]
- [[PowerAnalysis]]
- [[CongestionAnalysis]]

## Used by
- [[Routing]]
- [[Signoff]]
- [[STA]]

## Related
- [[ClockTreeSynthesis]]
- [[CTSFlow]]
- [[PostCTSOptimization]]
- [[STA]]
- [[Signoff]]
- [[PowerAnalysis]]
- [[CongestionAnalysis]]
