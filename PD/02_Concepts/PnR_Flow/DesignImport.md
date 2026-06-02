---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Bước đầu tiên của PnR flow trong Cadence Innovus — trước Floorplanning
used_by: [Floorplanning]
requires: [GateLevelNetlist, CellAbstract, MMMC, PhysicalConstraints, ITF]
chain: Chain_PnR_Flow
---
# DesignImport

## Definition
DesignImport là bước đầu tiên của PnR flow, gồm 7 sub-steps nhằm đọc, tích hợp, và kiểm tra toàn bộ inputs trước khi chuyển sang Floorplanning. Trong thực hành, bước này cần nạp đủ các nhóm input chính trước `init_design`: Netlist, MMMC/SDC, LEF/physical abstracts, và khai báo power/ground nets kèm physical constraints. Exit criterion của DesignImport: design meet timing với zero RC delays, constraints đầy đủ, không có errors.

**7 Sub-steps**:

| Sub-step                       | Nội dung                                                                        | Input                                                          |
| ------------------------------ | ------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 1. Netlist Import              | Đọc Gate-Level Netlist; xây dựng design hierarchy và connectivity database      | GateLevelNetlist (Verilog/VHDL)                                |
| 2. Cell Abstracts Import       | Đọc [[LEF]] files; xây dựng physical database của tất cả cells                  | CellAbstract (SC + Macro + Pad LEFs) + P&R Tech File + [[ITF]] |
| 3. Timing Constraints Import   | Đọc MMMC file; khởi tạo Library Sets, RC Corners, Delay Corners, Analysis Views | MMMC (TCL)                                                     |
| 4. Physical Constraints Import | Khai báo Power/Ground Net names; load IO placement và Block boundary            | IO Placement file (.io) + TCL scripts                          |
| 5. Initialize Design           | Tổng hợp tất cả inputs → khởi tạo unified design database                       | Tất cả inputs trên                                             |
| 6. Quality of Inputs Checks    | Kiểm tra tính hợp lệ và đầy đủ                                                  | P&R log file + Netlist + Constraints                           |
| 7. Quick Timing Checks         | Timing sanity check với zero RC                                                 | Design database                                                |

*Ghi chú:* các command names như `read_netlist`, `read_mmmc`, `read_physical`, `init_design`, `check_timing`, `report_analysis_coverage` trong tài liệu L5 nên được hiểu là **ví dụ theo flow minh họa**; tên lệnh cụ thể có thể khác giữa các tool/PnR environments.

Sau bước 5 (Initialize), PnR tool hiển thị design database ở trạng thái sơ khai với bốn thành phần: Modules (Core logic chưa có vị trí cố định), Floorplan boundary (nếu die size đã được set), IO Pads và Blocks/Blackboxes (hierarchical blocks hoặc undefined modules).

## Computed from

**Bước 6 — Quality of Inputs Checks** — các điều cần verify:

*Netlist quality:*
- Tất cả instances unique (không có duplicate)
- Không có empty modules → Black Box risk
- Hierarchy đầy đủ, không có unresolved references

*Timing Constraints quality:*
- False paths và multi-cycle paths chính xác và có chủ ý
- Không có sink points không có clock (unconstrained clock endpoints)
- Không có unconstrained timing endpoints
- Không có dropped constraints (SDC objects không resolve → tool silently drop, paths trở thành unconstrained mà không có warning)
- Không có missing I/O constraints

*Physical quality:*
- Tất cả cells trong Netlist có CellAbstract tương ứng trong LEF
- IO Pad instance names trong `.io` file match Netlist
- Die size hợp lệ (lớn hơn tổng area của tất cả cells)

**Bước 7 — Quick Timing Checks** — lý thuyết đằng sau mỗi loại check (go/no-go gate trước [[Floorplanning]]):

*Coverage check*: Báo cáo tỷ lệ timing endpoints đã được constrain theo từng check type (Setup, Hold, Recovery, Pulse Width,...). Output là bảng phân loại: Met / Violated / Untested. Mục tiêu: "Untested" = 0 cho tất cả check types quan trọng.

*Consistency check*: Kiểm tra nội tại của Timing Constraints — ví dụ: có clock nào được định nghĩa nhưng không drive bất kỳ sequential element nào không? Có clock uncertainty âm không? Có path constraints mâu thuẫn nhau không?

*Timing report với zero RC*: Chạy STA với Net Delay = 0 (chỉ có Cell Delay). Đây là "lý tưởng nhất" về wire — nếu design không meet timing ngay cả khi không có wire delays, thì không có PnR optimization nào có thể cứu được.

$$\text{Zero-RC Slack} = (T_c + \delta_{skew}) - (t_{cq} + t_{logic,max} + t_{setup} + \delta_{uncertainty})$$

với $t_{logic,max} = \sum \text{CellDelay}_i$ (không có $\text{NetDelay}$). Negative zero-RC Slack = vấn đề Synthesis, phải fix trước khi tiếp tục PnR. Với ideal clock: δ_skew = 0 (pre-CTS, ideal clock)

Nếu zero-RC timing fail, hoặc timing coverage cho thấy còn nhiều "Untested" paths ở các check types quan trọng, design không nên proceed sang [[Floorplanning]] cho đến khi root cause được xử lý.

*Slack Histogram*: Phân phối của tất cả Slack values qua tất cả timing paths. WNS = most negative Slack; TNS = sum of all negative Slacks. Histogram cho phép estimate overall timing closure health: phần lớn paths nằm ở đâu? Có long tail của violations không?

## Constrains
- **[[Floorplanning]]**: chỉ bắt đầu khi DesignImport pass hoàn toàn (zero-RC timing clean + constraints coverage = 100%)

## Requires
- [[GateLevelNetlist]] — connectivity database
- [[CellAbstract]] — physical database (qua LEF)
- [[MMMC]] — timing analysis framework
- [[PhysicalConstraints]] — physical boundary và IO placement
- [[ITF]] — embedded trong MMMC RC Corners

## Used by
- [[Floorplanning]] — step tiếp theo trong PnR flow; nhận design database đã validated

## Key insight
[USER REVIEW — draft suggestion]: Zero-RC timing sanity check là "gate" quan trọng nhất của Import và là tín hiệu sớm nhất, rẻ nhất để phát hiện vấn đề Synthesis. Nếu fail ở đây, quay lại Synthesis ngay — đừng tiếp tục Floorplan. Timing Coverage check (% Untested) là metric thứ hai quan trọng: 13% Untested trong ExternalDelay checks (như ví dụ trong slide) nghĩa là 13% của I/O paths không được verify bởi STA — đây là blind spot có thể dẫn đến silicon failure. DesignImport thường chiếm 1–2 ngày trong schedule của một tape-out — không phải do runtime của tool, mà do time cần để debug SDC errors, resolve Black Boxes, và align IO constraints với Package team.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ 7 sub-steps: Netlist → Cell Abstracts → Timing Constraints → Physical Constraints → Init → QoI Check → Timing Check
→ Exit criteria: zero-RC WNS ≥ 0 · TNS ≥ 0 · constraint coverage = 100% · no errors
→ Inputs: [[GateLevelNetlist]] · [[CellAbstract]] · [[MMMC]] · [[PhysicalConstraints]] · [[ITF]]
→ Output: validated design database → ready for [[Floorplanning]]
→ Key concepts: Black Box · Unconstrained endpoint · Zero-RC timing · Slack Histogram
→ Cùng nhóm: [[Floorplanning]] · [[GateLevelNetlist]] · [[MMMC]] · [[PhysicalConstraints]]