---
tags: [concept, cell-library, pnr-flow]
group: Cell Library
defined_in: Standard Cell Library; inserted during row/core boundary preparation
used_by: [Floorplanning, Placement, Signoff]
requires: [StandardCell, Row]
chain: Chain_LEF_to_PnR
---
# EndCapCell

## Definition
EndCapCell là một **physical-only cell** được đặt ở đầu/cuối của các [[Row]] standard-cell, hoặc tại vùng biên liên quan đến kết thúc row trong core. Mục tiêu là tạo termination hợp lệ cho row-edge ở mức physical/electrical convention.

## Why EndCapCell exists
Ở mức khái niệm, EndCapCell giúp:
- “khóa” biên row để cấu trúc hàng standard cell có điểm kết thúc hợp lệ,
- bảo vệ/tổ chức row-edge behavior theo library convention,
- hỗ trợ tính nhất quán hình học và tính kiểm chứng ở downstream stages.

Chi tiết implementation cụ thể theo từng library/process có thể khác nhau. **[Needs verification]**

## Boundary Cell alias note
**EndCapCell is sometimes also described as a Boundary Cell in row-edge context.**

Trong card này, ưu tiên tên **EndCapCell** để nhấn mạnh phạm vi “row termination”, tránh mơ hồ với boundary ở cấp core/block/IO.

## Relation to Row / StandardCell
EndCapCell gắn trực tiếp với cấu trúc [[Row]] và ecosystem của [[StandardCell]]. Dù không thực hiện logic function, nó là phần tử triển khai vật lý giúp row closure hợp lệ trước/sau placement optimization.

## Distinction from TapCell / FillerCell / DecapCell
- **vs [[TapCell]]**: TapCell xử lý well/substrate bias (Latch-up prevention), không phải row-end termination.
- **vs FillerCell**: FillerCell chủ yếu lấp khoảng trống để duy trì continuity cục bộ; khác mục tiêu so với end-cap tại row edge. **[Needs verification]**
- **vs DecapCell**: DecapCell phục vụ decoupling/ổn định nguồn; không phải cell kết thúc row.

(DecapCell/FillerCell sẽ được tách card riêng ở batch sau.)

## Constrains
- Placement của EndCapCell phụ thuộc row/core boundary conventions của library.
- Không suy diễn spacing/geometry numbers nếu chưa có process rule cụ thể. **[Needs verification]**

## Requires
- [[StandardCell]] — EndCapCell thuộc thư viện cell vật lý.
- [[Row]] — vị trí chèn bám theo đầu/cuối Standard Cell Row.

## Used by
- [[Floorplanning]] — chuẩn bị row/core boundary theo physical-only cell strategy.
- [[Placement]] — thực thi/legalize vị trí end-cap trong row context.
- [[Signoff]] — kiểm tra tính hợp lệ của boundary/row termination theo design rules.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Liên quan: [[StandardCell]] · [[Row]] · [[Floorplanning]]
→ Phân biệt: [[TapCell]] · FillerCell (deferred) · DecapCell (deferred)
