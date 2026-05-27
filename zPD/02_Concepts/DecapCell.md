---
tags: [concept, cell-library, pnr-flow]
group: Cell Library
defined_in: Standard Cell Library; inserted during physical implementation / power integrity optimization
used_by: [Floorplanning, Placement, PDN, IRDrop, Signoff]
requires: [StandardCell, PDN]
chain: Chain_LEF_to_PnR
---
# DecapCell

## Definition
DecapCell là một **physical-only standard cell** cung cấp **local decoupling capacitance** trong vùng digital implementation. DecapCell không thực hiện logic function; vai trò chính là bổ sung hiệu ứng Decoupling Capacitor cục bộ trong fabric của standard cells.

## Why DecapCell exists
Ở mức khái niệm, DecapCell giúp:
- cải thiện local power stability,
- giảm mức **Voltage Droop** cục bộ khi switching activity tăng,
- hỗ trợ giảm rủi ro **Dynamic IR Drop** theo vùng.

DecapCell vì vậy là phần hỗ trợ power integrity trong physical implementation, không phải cell xử lý dữ liệu.

## Relation to PDN and IR Drop
DecapCell liên quan trực tiếp đến bối cảnh [[PDN]] và [[IRDrop]]:
- PDN tạo hạ tầng phân phối nguồn (rings/straps/rails),
- DecapCell hỗ trợ ổn định nguồn cục bộ trong hạ tầng đó.

DecapCell **không thay thế** thiết kế PDN kiến trúc. Nếu PDN topology yếu hoặc phân phối nguồn không đủ, chỉ chèn DecapCell sẽ không giải quyết triệt để vấn đề.

## Distinction from other physical-only cells
- **vs [[TieCell]]**: TieCell tạo constant logic tie-off, không nhằm decoupling nguồn.
- **vs [[TapCell]]**: TapCell phục vụ well/substrate bias và latch-up prevention, không phải decoupling capacitance.
- **vs [[EndCapCell]]**: EndCapCell xử lý row-end termination, không phục vụ ổn định nguồn cục bộ.
- **vs [[FillerCell]]**: FillerCell dùng để gap filling / continuity của row, không phải mục tiêu Voltage Droop mitigation.

## Flow / PDK dependence
Chính sách chèn DecapCell (vị trí, mật độ, thời điểm, ưu tiên theo vùng) là **flow/tool/library/PDK dependent**. **[Needs verification]**

Không dùng các giá trị capacitance, density target, spacing rule, hay insertion interval khi chưa có guideline process/tool tương ứng. **[Needs verification]**

## Constrains
- DecapCell phải được hiểu là lớp hỗ trợ local PG behavior, không phải giải pháp thay cho PDN architecture.
- Insertion methodology cần bám theo quy tắc library/PDK/project-specific. **[Needs verification]**

## Requires
- [[StandardCell]] — DecapCell thuộc standard cell library.
- [[PDN]] — vai trò của DecapCell chỉ có ý nghĩa trong context phân phối nguồn.

## Used by
- [[Floorplanning]] — xác định chiến lược power integrity mức khái niệm.
- [[Placement]] — thực thi chèn/legalize DecapCell theo legal rows và constraints.
- [[PDN]] — liên kết trực tiếp tới ổn định nguồn cục bộ trong kiến trúc PG.
- [[IRDrop]] — liên quan tới giảm rủi ro droop động theo vùng.
- [[Signoff]] — kiểm tra mức độ hợp lệ/convergence của quality power integrity.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Cùng nhóm: [[StandardCell]] · [[PDN]] · [[IRDrop]] · [[FillerCell]]
→ Phân biệt: [[TieCell]] · [[TapCell]] · [[EndCapCell]]
