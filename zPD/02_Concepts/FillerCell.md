---
tags: [concept, cell-library, pnr-flow]
group: Cell Library
defined_in: Standard Cell Library; inserted during placement legalization / physical finishing
used_by: [Placement, Floorplanning, Signoff]
requires: [StandardCell, Row]
chain: Chain_LEF_to_PnR
---
# FillerCell

## Definition
FillerCell là một **physical-only standard cell** dùng để lấp các khoảng trống giữa các standard cells đã placed trong cùng một [[Row]]. FillerCell không thực hiện logic function; mục tiêu chính là hoàn thiện tính liên tục vật lý của Standard Cell Row.

## Why FillerCell exists
Ở mức khái niệm, FillerCell giúp:
- fill gap giữa các placed cells,
- duy trì **row physical continuity**,
- hỗ trợ **Well/Implant Continuity**,
- hỗ trợ tính đầy đủ rule-completeness cho layout ở mức physical finishing.

## Relation to Row / StandardCell
FillerCell gắn trực tiếp với [[Row]] và [[StandardCell]] ecosystem:
- placement tạo ra các gap nhỏ theo legal sites,
- filler insertion dùng để hoàn thiện vùng row sau các bước place/opt.

Cách chọn loại filler và thứ tự chèn phụ thuộc library và tool policy. **[Needs verification]**

## Distinction from other physical-only cells
- **vs [[EndCapCell]]**: EndCapCell xử lý row-end/boundary termination; FillerCell xử lý các gap nội bộ giữa cells.
- **vs [[TapCell]]**: TapCell phục vụ well/substrate bias; FillerCell không thay vai trò bias taps.
- **vs [[TieCell]]**: TieCell tạo constant logic tie-off; FillerCell không tạo logic constant.
- **vs [[DecapCell]]**: DecapCell phục vụ local decoupling capacitance; FillerCell phục vụ continuity/gap fill.

## Flow / PDK dependence
Chính sách filler selection/insertion (mapping cell widths, priority rules, legalization behavior) là **library/tool/PDK dependent**. **[Needs verification]**

Không đưa các quy tắc geometry, spacing, filler-order cụ thể, hay insertion interval khi chưa có process guideline tương ứng. **[Needs verification]**

## Constrains
- FillerCell dùng cho gap-filling và continuity mục tiêu, không thay thế EndCap/Tap/Tie/Decap roles.
- Rule chi tiết về filler deployment phải theo flow/library/PDK của project. **[Needs verification]**

## Requires
- [[StandardCell]] — FillerCell thuộc standard cell library.
- [[Row]] — vị trí chèn dựa trên cấu trúc legal Standard Cell Row.

## Used by
- [[Placement]] — chèn/legalize filler trong physical finishing của cell rows.
- [[Floorplanning]] — cung cấp bối cảnh row-based implementation cho downstream insertion.
- [[Signoff]] — kiểm tra tính đầy đủ vật lý và rule-clean closure ở cuối luồng.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Cùng nhóm: [[StandardCell]] · [[Row]] · [[EndCapCell]] · [[DecapCell]]
→ Phân biệt: [[TapCell]] · [[TieCell]]
