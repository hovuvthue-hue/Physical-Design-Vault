---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: N/A — computed quantity bởi STA tool; reported trong timing reports của Cadence Tempus / Synopsys PrimeTime
used_by: [STA, LogicSynthesis, Placement, ClockTreeSynthesis, Routing, PostRouteOptimization, Signoff]
requires: [STA, SDC, CellDelay, NetDelay]
chain: Chain_STA_Basics
---
# Slack

## Definition
Slack là đại lượng định lượng duy nhất biểu diễn mức độ đáp ứng (hoặc vi phạm) timing constraints của một timing path, được tính bằng hiệu số giữa Required Arrival Time (RAT — thời điểm deadline) và Arrival Time (AT — thời điểm tín hiệu thực sự đến): `Setup Slack = RAT − AT`. Slack dương (positive) nghĩa là tín hiệu đến trước deadline — timing Met; Slack âm (negative) nghĩa là tín hiệu đến sau deadline — Timing Violation; Slack = 0 là ranh giới nguy hiểm không có margin. Slack là output trực tiếp của STA và là metric cốt lõi mà toàn bộ PD flow tối ưu hóa hướng đến.

## Computed from
Slack được tính tại mỗi Endpoint sau khi STA hoàn thành Forward và Backward Propagation:

**Setup Slack (Max Delay Check):**
AT = t_launch_clock + t_cq + Σ Stage_Delay(i) [Forward] 
RAT = t_capture_clock + T_period − t_setup − t_uncertainty [Backward] 
Setup Slack = RAT − AT

**Hold Slack (Min Delay Check):**
AT_min = t_launch_clock + t_ccq + Σ Stage_Delay_min(i) [Forward, MIN paths] 
RAT_hold = t_capture_clock + t_hold [Backward] 
Hold Slack = AT_min − RAT_hold

Trong thực tế với [[ClockSkew]] và OCV Margin đầy đủ:
- Setup: `Slack = (T_c + δ_skew) − (t_cq + t_logic_max + t_setup + δ_margin)`
- Hold: `Slack = (t_ccq + t_logic_min − δ_margin) − (t_hold + δ_skew)`

Ba macro-metrics tổng hợp từ Slack của tất cả paths:
- **WNS (Worst Negative Slack)**: `min(Slack)` toàn chip → quyết định max operating frequency thực tế; WNS = 0 là target tối thiểu trước Tape-out.
- **TNS (Total Negative Slack)**: `Σ(negative Slacks)` → phản ánh tổng khối lượng ECO cần giải quyết; TNS lớn → violations phân tán rộng.
- **FEP (Failing Endpoints)**: số Endpoints có Slack âm → phản ánh diện rộng của timing violations; FEP cao → nhiều paths cần optimization, không chỉ worst path. Phân biệt: WNS = severity của violation tệ nhất; FEP = số lượng endpoints bị ảnh hưởng.

## Constrains
- **[[SetupTime]]**: Setup Slack âm → data path quá chậm → Setup violation; fix bằng cách tăng AT room (upsize cells, shorten wires, add pipeline stage) hoặc tăng RAT (relax clock period — nhưng làm giảm chip performance)
- **[[HoldTime]]**: Hold Slack âm → data path quá nhanh → Hold violation; fix bằng cách insert delay buffers trên data path để tăng AT_min; Hold violations không thể fix bằng cách giảm tần số (T_c không xuất hiện trong Hold Slack equation)
- **[[Signoff]]**: Tất cả Slack values phải ≥ 0 ở tất cả MMMC corners/modes trước Tape-out — đây là hard criterion; WNS < 0 ở bất kỳ corner nào → block Tape-out

## Requires
- [[STA]] — Slack là output của STA computation; không thể có Slack value nếu không chạy STA với đầy đủ inputs
- [[SDC]] — T_period (từ `create_clock`) và t_uncertainty (từ `set_clock_uncertainty`) trong Slack equation đến từ SDC; Slack values chỉ có nghĩa tương đối với SDC đang dùng
- [[CellDelay]] · [[NetDelay]] — cấu thành Σ Stage_Delay trong AT computation; quality của Cell Delay models (NLDM vs CCS) và Net Delay accuracy (WLM vs SPEF) ảnh hưởng trực tiếp đến độ tin cậy của Slack values

## Used by
- [[STA]] — compute và report Slack tại mỗi Endpoint; sort paths theo Slack để generate WNS/TNS summary; `-max_paths N` option report N paths có Slack âm sâu nhất
- [[LogicSynthesis]] — synthesis engine minimize TNS bằng cách iterate over negative-Slack paths và swap cells hoặc restructure logic; WNS là stopping criterion của optimization loop
- [[Placement]] — timing-driven placement weight placement cost function theo Slack per path; paths với Slack âm sâu được ưu tiên đặt cells gần nhau hơn
- [[ClockTreeSynthesis]] — post-CTS STA generate lại Slack với propagated clock; [[ClockSkew]] và [[ClockLatency]] thực tế có thể làm dịch chuyển cả Setup/Hold Slack so với pre-CTS ideal clock assumption
- [[Routing]] — timing-driven routing ưu tiên route paths có Slack âm trên lower-resistance metal layers; post-route STA với SPEF generate final Slack values cho Signoff
- [[PostRouteOptimization]] — dùng post-route Slack sau khi annotate SPEF để ưu tiên setup/hold cleanup và kiểm tra regression do chỉnh sửa hậu route
- [[Signoff]] — Timing Signoff pass criterion: WNS ≥ 0 và TNS = 0 ở tất cả Analysis Views; WNS và TNS là hai con số kỹ sư PD report cho team lead mỗi ngày

## Key insight
[USER REVIEW — draft suggestion]: Slack là ngôn ngữ chung của toàn bộ PD team — mọi quyết định kỹ thuật từ synthesis đến signoff đều được diễn đạt qua Slack. Điều quan trọng cần hiểu: Slack là một con số tương đối, không tuyệt đối — "Slack = +0.05ns" không có nghĩa là chip an toàn nếu SDC đang dùng clock uncertainty quá nhỏ hoặc OCV derating chưa được áp dụng. Thực tế trong industry: target thường là WNS ≥ 0 với đầy đủ margin đã được folded vào SDC (jitter, OCV, aging derating); một design có WNS = 0 chính xác sẽ thường xuyên fail silicon test. Phân biệt quan trọng: Setup Slack phụ thuộc vào T_period (có thể cứu bằng cách giảm tần số sau silicon), còn Hold Slack hoàn toàn độc lập với T_period — Hold violation là silicon kill, không có workaround.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Formula: RAT − AT (Setup) · AT_min − RAT_hold (Hold)
→ Macro-metrics: WNS (Worst Negative Slack) · TNS (Total Negative Slack)
→ Depends on: [[STA]] · [[SDC]] · [[CellDelay]] · [[NetDelay]] · [[SetupTime]] · [[HoldTime]] · [[ClockSkew]]
→ Reported by: Cadence Tempus · Synopsys PrimeTime
→ Cùng nhóm: [[STA]] · [[SDC]] · [[CellDelay]] · [[NetDelay]] · [[StageDelay]] · [[SetupTime]] · [[HoldTime]]