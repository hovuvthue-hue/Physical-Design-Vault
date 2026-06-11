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

Kết quả CTS phải được review để đảm bảo các yêu cầu về timing, electrical, physical implementation, routability, và clock quality đã được thỏa mãn trước khi chuyển sang routing và signoff stages.

## CTS Deliverables

CTS tạo ra các đầu ra là cơ sở cho quality review:

- Optimized gate-level netlist (cập nhật với clock buffer/inverter được chèn vào)
- Clock-routed DEF database
- Timing reports
- Clock tree quality reports
- Power analysis reports

## 1. Timing and Electrical Checks

**Timing Checks:**
- Setup và Hold timing violations phải được loại bỏ hoặc giảm xuống mức tối thiểu.
- WNS (Worst Negative Slack) và TNS (Total Negative Slack) phải gần bằng 0.
- Số lượng failing endpoints (FEP) phải ở mức tối thiểu.

**Electrical and DRV Quality:**
- Tất cả electrical constraint violations (max transition/capacitance/fanout) phải được xử lý hoặc giảm thiểu, **bao gồm cả clock nets**.

Ngưỡng pass/fail cụ thể là flow-dependent. [Needs verification]

## 2. Placement and Routability Checks

**Placement Legality Checks:**
- Tất cả cells phải được legally placed.
- Không có cells nào bị overlap, unplaced, hoặc vi phạm placement rules.
- Placement legalization đảm bảo: valid cell alignment, row compliance, proper spacing, manufacturable placement.
- Placement density phải nằm trong giới hạn chấp nhận được để tránh severe congestion.

**Routability Checks:**
- Design phải duy trì khả năng routable.
- Target routability điển hình:
  - Overflow < 1%
  - Hotspot count < 100

Mức pass/fail cụ thể phụ thuộc tool metric và methodology. [Needs verification]

## 3. Power Consumption Checks

Clock network thường là nguồn đóng góp dynamic power đáng kể vì clock signals toggle liên tục. CTS phải thỏa mãn power targets tổng thể trong khi tối thiểu hóa tác động của clock distribution.

**Power Checks After CTS:**
- Total power consumption — bao gồm dynamic power, leakage power, và internal power — phải nằm trong design targets.
- Clock networks là major contributors của dynamic switching power vì clock signals toggle liên tục.
- CTS optimization phải minimize không cần thiết: clock buffer insertion, clock switching activity, clock tree depth, và clock capacitance.

Kết quả định lượng nên được đọc cùng [[PowerAnalysis]] trong đúng context stage/corner. [Needs verification]

## 4. Clock Tree Quality Checks

Các chỉ số clock quality cốt lõi:

- **[[ClockSkew]]** phải nằm trong target limits.
- **Clock insertion delay** phải được tối thiểu hóa hoặc kiểm soát theo CTS targets trong khi vẫn duy trì balanced clock distribution.
- **[[ClockLatency]]** phải cân bằng giữa các clock sinks trong cùng một skew group.
- **Clock transition quality** ([[Slew]]) phải thỏa mãn maximum transition và electrical constraint requirements.

Các chỉ số này cần được xem đồng thời thay vì tách rời, vì tối ưu một chiều có thể làm xấu chiều khác trong downstream closure.

### Phân biệt Insertion Delay và Clock Latency

Trong context Clock Tree Quality Checks, cần phân biệt:

- **Insertion Delay**: fixed CTS estimate — không thay đổi sau khi clock tree đã được build; là snapshot từ thời điểm tree construction.
- **[[ClockLatency]]**: live STA value — cập nhật mỗi khi extraction được chạy sau routing; phản ánh trạng thái extracted hiện tại của design.

Cả hai đo cùng một đường vật lý nhưng ở các thời điểm khác nhau. Sau routing và extraction, Clock Latency cập nhật theo wire RC thực tế, trong khi Insertion Delay vẫn giữ nguyên. Tại Signoff, Clock Latency là con số phản ánh thực tế chính xác hơn.

## CTS Checklist

Tổng hợp các hạng mục cần pass trước khi chuyển sang Routing:

**Timing Checks:**
- Không có setup violations hoặc chỉ còn minimal setup violations.
- Không có hold violations hoặc chỉ còn minimal hold violations.
- WNS và TNS gần bằng 0.
- FEP ở mức tối thiểu.

**Electrical Checks:**
- Không có DRV violations trên cả signal nets và clock nets.

**Placement and Routability Checks:**
- Tất cả cells legally placed và fully legalized.
- Congestion overflow và hotspot levels trong acceptable limits.

**Power Consumption Checks:**
- Total power đáp ứng design targets.
- Clock network power được tối ưu.

**Clock Tree Quality Checks:**
- Clock insertion delay đáp ứng CTS targets.
- Clock skew trong target limits.
- Clock transition quality đáp ứng electrical constraints.

## Debug Aids — Clock Tree Debugger (CTD)

CTD (Clock Tree Debugger) cung cấp graphical views để inspect, analyze, và debug clock tree structure, timing, và balance. Hai views chính:

**Clock Tree Layout View:** Hiển thị vị trí vật lý của clock tree cells và routing trên die. Engineer có thể trace clock path từ source đến sinks và identify physical issues như blockages hoặc unusual routing detours.

**Clock Insertion Delay View:** Plots mỗi clock tree cell theo insertion delay trên trục Y — buffers/inverters (vàng), ICGs (xanh lá), flip-flops (đỏ) từ trên xuống dưới. Các red markers (flip-flops) tụ sát nhau ở phía dưới chỉ ra clock tree được cân bằng tốt với minimal skew; spread rộng lập tức flag skew problem cần debug.

**Cross-View Navigation:** Chọn một cell hoặc path trong một view sẽ highlight elements tương ứng trong view kia, cho phép locate và investigate problem areas trên die nhanh chóng.

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