---
tags: [concept, pnr-flow, cts, optimization]
group: PnR Flow
defined_in: Post-CTS Optimization
used_by: [ClockTreeSynthesis, Routing, STA, Signoff]
requires: [ClockTreeSynthesis, CTSFlow, CTSOptimization, STA, Slack, DRVFixing]
chain: Chain_PnR_Flow
---
# PostCTSOptimization

## Definition
**PostCTSOptimization** là giai đoạn tối ưu hóa sau [[ClockTreeSynthesis]], khi clock đã chuyển từ giả định ideal sang bối cảnh propagated clock trong phân tích [[STA]].

Ở giai đoạn này, hiệu ứng clock-tree thực tế (arrival, skew, transition, loading) bắt đầu tác động trực tiếp vào kết quả timing/DRV, nên hoạt động cleanup phản ánh gần thực tế implementation hơn so với trước CTS.

## Position in flow
Trong flow vật lý, PostCTSOptimization thường nằm sau [[CTSFlow]] và trước khi thiết kế đi sâu vào [[Routing]] chi tiết và kiểm tra readiness cho [[Signoff]].

Nói ngắn gọn: đây là lớp closure trung gian để ổn định QoR sau khi clock tree đã được xây dựng.

## Difference from PreCTSOptimization / CTSOptimization / Post-Conditioning
- **Khác [[PreCTSOptimization]]**: Pre-CTS làm việc khi clock context còn lý tưởng/ước lượng; PostCTSOptimization làm việc với hiệu ứng propagated clock.
- **Khác [[CTSOptimization]]**: CTSOptimization là các kỹ thuật tối ưu nằm bên trong hoặc bám sát tiến trình dựng clock tree; PostCTSOptimization là bước cleanup downstream rộng hơn sau khi CTS đã hoàn tất.
- **Khác Post-Conditioning trong [[CTSFlow]]**: Post-Conditioning là pha nội bộ của CTS engine; PostCTSOptimization là pha closure cấp flow sau CTS.

## Main cleanup targets
Các mục tiêu thường gặp ở mức concept:
- setup recovery dựa trên kết quả post-CTS [[STA]]
- hold recovery khi skew/latency thực bắt đầu lộ rõ
- repair electrical DRV thông qua [[DRVFixing]]
- transition/[[Slew]] cleanup trên các net/cell nhạy cảm
- capacitance/fanout cleanup để giảm rủi ro vi phạm tiếp diễn
- skew refinement có kiểm soát để cải thiện tính ổn định timing

Thứ tự ưu tiên các mục tiêu trên phụ thuộc flow/tool/library cụ thể. [Needs verification]

## Setup vs hold recovery trade-off
PostCTSOptimization là nơi trade-off giữa setup và hold trở nên rõ:
- sửa hold có thể làm xấu setup ở path liên quan,
- cải thiện setup có thể mở ra hold risk mới ở nhánh khác.

Vì vậy, các vòng tối ưu thường mang tính incremental và cần theo dõi đồng thời nhiều nhóm path thay vì fix cục bộ một vi phạm đơn lẻ.

## DRV and transition cleanup
Sau CTS, các thao tác như thêm buffer, đổi kích thước cell, hoặc tái cân bằng tải có thể giúp dọn DRV/transition; nhưng đồng thời có thể làm tăng area, power và nhu cầu tài nguyên route.

Tiêu chí chấp nhận mức DRV/transition trước khi chuyển hẳn sang downstream stage là flow-dependent. [Needs verification]

## Incremental optimization
PostCTSOptimization thường không phải một pass duy nhất mà là chuỗi vòng lặp incremental:
1. chạy phân tích timing/electrical trong bối cảnh post-CTS,
2. áp dụng chỉnh sửa có kiểm soát,
3. đánh giá lại ảnh hưởng chéo giữa setup, hold, DRV, power và routability.

Chiến lược lặp, mức hội tụ, và điều kiện dừng phụ thuộc tool flow cụ thể. [Needs verification]

## Requires
- [[ClockTreeSynthesis]]
- [[CTSFlow]]
- [[CTSOptimization]]
- [[STA]]
- [[Slack]]
- [[DRVFixing]]

## Used by
- [[ClockTreeSynthesis]]
- [[Routing]]
- [[STA]]
- [[Signoff]]

## Related
- [[PreCTSOptimization]]
- [[CTSFlow]]
- [[CTSOptimization]]
- [[ClockTreeSynthesis]]
- [[DRVFixing]]
- [[STA]]
- [[Slack]]
- [[Routing]]
- [[Signoff]]
