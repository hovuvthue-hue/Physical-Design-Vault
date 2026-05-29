---
tags: [concept, file-format, lef-geometry]
group: File Formats
defined_in: Cadence Innovus / Synopsys ICC2 — Design Exchange Format; ASCII text; companion format của LEF
used_by: [Floorplanning, Placement, Routing, ParasiticExtraction, Signoff]
requires: [GateLevelNetlist, LEF]
chain: Chain_LEF_to_PnR
---
# DEF

## Definition
DEF (Design Exchange Format) là file text ASCII chứa thông tin instance-specific của một design cụ thể — tức là dữ liệu thay đổi theo từng design, ngược lại với LEF chứa dữ liệu cố định của technology và library. DEF và LEF là cặp format bổ sung cho nhau: LEF định nghĩa "NAND2X1 trông như thế nào và rộng bao nhiêu", còn DEF định nghĩa "instance U45 của NAND2X1 được đặt ở tọa độ (1200, 3400) với orientation N". DEF là format snapshot của design state tại mỗi checkpoint trong PnR flow — Import DB, Floorplan DB, Placement DB, CTS DB, Route DB đều được lưu dưới dạng DEF.

## Computed from
DEF được generate và update bởi PnR tool (Cadence Innovus, Synopsys ICC2) tại mỗi stage. Nội dung DEF tăng dần theo flow:

**Post-Import DEF**: UNITS, DIEAREA (die boundary), COMPONENTS (cell instances từ Netlist nhưng chưa có coordinates), NETS (connectivity từ Netlist), PINS (I/O port locations nếu có sẵn từ physical constraints).

**Post-Floorplan DEF**: Bổ sung ROWS (row definitions dựa trên SITE từ Tech LEF), TRACKS (routing grid definitions), Macro placements với fixed coordinates, SPECIALNETS (power/ground net routes của PDN).

**Post-Placement DEF**: Tất cả COMPONENTS có coordinates (x, y) và orientation (N/S/E/W/FN/FS/FE/FW) đã được legalized.

**Post-Route DEF**: Tất cả NETS có routing geometry — layer, width, coordinates của từng wire segment và via.

## Constrains
- **[[Floorplanning]]**: DEF chứa DIEAREA (kích thước chip), ROWS (placement rows), và Macro/IO placements — đây là output của Floorplanning và là input constraints cho Placement; floorplan quality được capture hoàn toàn trong DEF
- **[[Placement]]**: Post-placement DEF với cell coordinates là input bắt buộc cho CTS — CTS tool đọc DEF để biết vị trí chính xác của tất cả clock sinks
- **[[Routing]]**: Post-CTS DEF với fixed clock tree topology là baseline cho signal routing; Router update DEF bằng cách add wire geometry cho tất cả signal nets
- **[[ParasiticExtraction]]**: Post-route DEF (hoặc GDS) chứa wire geometry là primary input cho RC extraction; extraction tool map extracted parasitics về đúng net names trong DEF

## Requires
- [[GateLevelNetlist]] — cell instances và net connectivity trong Netlist là source data để populate COMPONENTS và NETS sections trong DEF; không có Netlist, DEF không có nội dung design
- [[LEF]] — DEF references MACRO names (từ Macro LEF) và SITE names (từ Tech LEF) trong COMPONENTS và ROWS sections; DEF phải được read cùng với LEF để PnR tool có thể resolve geometric data

## Used by
- [[Floorplanning]] — reads và writes DEF; Floorplanning là bước đầu tiên add physical information (ROWS, TRACKS, Macro positions) vào DEF
- [[Placement]] — reads Floorplan DEF, writes Placement DEF với legalized cell coordinates; DEF là primary working format của Placement engine
- [[Routing]] — reads Placement/CTS DEF, writes Route DEF với full wire geometry; Route DEF là near-final layout representation trước khi export GDS
- [[ParasiticExtraction]] — reads Route DEF (hoặc GDS) để extract wire geometry; net names trong DEF map extracted RC values vào correct net context cho SPEF
- [[Signoff]] — LVS tool reads DEF/GDS và compares with Netlist; DRC tool reads DEF/GDS và checks design rules; Route DEF là basis cho all signoff checks

## Key insight
[USER REVIEW — draft suggestion]: DEF là "sổ tay trạng thái" của design trong PnR flow — mỗi lần lưu DEF là một checkpoint snapshot. Điều này làm cho DEF cực kỳ quan trọng trong debugging: nếu Routing fail hoặc timing bị tệ đi sau một step, engineer có thể load DEF từ checkpoint trước đó và restart từ điểm đó thay vì chạy lại toàn bộ flow. Thực tế quan trọng: DEF và LEF phải được đọc cùng nhau — DEF chỉ chứa tên của MACRO và SITE (ví dụ "COMPONENT U45 NAND2X1"), còn geometry thực sự (NAND2X1 rộng bao nhiêu, Pin ở đâu) phải được lookup từ LEF. Đây là lý do import step trong PnR tool luôn yêu cầu cả LEF lẫn DEF cùng lúc.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Companion format: [[LEF]] — LEF = library/technology data; DEF = design instance data
→ Updated at each stage: Import DB → Floorplan DB → Placement DB → CTS DB → Route DB
→ Sources connectivity from: [[GateLevelNetlist]]
→ Contains geometry for: [[Floorplanning]] results · [[Placement]] results · [[Routing]] results
→ Basis for: [[ParasiticExtraction]] · [[Signoff]] checks
→ Cùng nhóm: [[LEF]] · [[LIB]] · [[SPEF]] · [[GDS]]