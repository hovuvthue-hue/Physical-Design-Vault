---
tags: [concept, pd-framework]
group: PD Framework
defined_in: N/A — framework khái niệm, không thuộc file format hay tool cụ thể
used_by: [Floorplanning, Placement, CTS, Routing, Signoff]
requires: [N/A]
chain: N/A — cross-cutting concept, áp dụng cho toàn bộ PD flow
---
# PPARMP

## Definition
PPARMP là framework 6 mục tiêu tối ưu hóa của Physical Design, viết tắt của: **P**erformance (tốc độ xử lý, đáp ứng Timing constraints), **P**ower (tiêu thụ điện năng — dynamic và static), **A**rea (diện tích chip), **R**eliability (hoạt động đúng trong mọi điều kiện vận hành), **M**anufacturability (tối ưu cho quy trình sản xuất, tăng yield), **P**roductivity (đạt design closure nhanh, giảm time-to-market). Ba mục tiêu đầu (PPA) là core optimization targets; ba mục tiêu sau (RMP) là constraints phải đáp ứng.

## Computed from
PPARMP không được tính toán — đây là framework định hướng quyết định kỹ thuật. Tuy nhiên, 6 mục tiêu có quan hệ trade-off định lượng được:
- **Performance vs. Power**: tăng clock frequency → tăng dynamic power (P_dynamic = α × C × V² × f)
- **Performance vs. Area**: improve timing bằng cách upsize cells → tăng cell area
- **Power vs. Area**: insert power gating / clock gating → thêm logic overhead → tăng area
- Không có điểm tối ưu tuyệt đối — mọi design đều là một điểm trade-off trên không gian 6 chiều này, được xác định bởi application requirements (ví dụ: mobile chip ưu tiên Power; server CPU ưu tiên Performance)

## Constrains
- **Floorplanning**: Area target xác định Core area budget; Power target xác định PDN requirements (IR Drop budget)
- **Placement**: Performance target drive timing-driven placement; Power target drive clock gating cell placement
- **CTS**: Performance và Power target cùng lúc constrain Clock Tree design — Skew nhỏ (Performance) nhưng buffer count ít (Power)
- **Routing**: Manufacturability target enforce DRC rules và metal density rules; Reliability target enforce EM current density limits
- **Signoff**: tất cả 6 targets phải được verify pass trước Tape-out — mỗi target tương ứng với ít nhất một signoff check

## Requires
N/A — PPARMP là framework định nghĩa goals, không phụ thuộc vào concept kỹ thuật upstream nào. Nó được derive từ product requirements (MRD/PRD) ở cấp độ system specification, trước khi Physical Design bắt đầu.

## Used by
- [[Floorplanning]] — Area và Power targets là inputs trực tiếp cho Core area sizing và PDN design
- [[Placement]] — Performance (Timing) target drive placement optimization objective function
- [[ClockTreeSynthesis]] — Performance (Skew budget) và Power (clock power budget) targets constrain CTS optimization
- [[Routing]] — Manufacturability (DRC) và Reliability (EM) targets là hard constraints trong Routing
- [[Signoff]] — mỗi signoff check verify ít nhất một trong 6 PPARMP targets: Timing Signoff (P+P), DRC/LVS (M), IR Drop/EM (R+P), Power analysis (P)

## Key insight
[USER REVIEW — draft suggestion]:
PPARMP hữu ích nhất khi dùng như một checklist để hiểu tại sao một quyết định kỹ thuật được đưa ra — bất kỳ lựa chọn nào trong PD flow (cell sizing, buffer insertion, Macro placement, metal layer assignment) đều có thể được phân tích qua lens của 6 targets này. Thực tế trong industry: không bao giờ có thể tối ưu cả 6 targets đồng thời — kỹ sư PD phải biết rõ application của chip đang làm để biết target nào được ưu tiên, target nào có thể trade-off.

## Related
→ Áp dụng cho toàn bộ: [[Chain_PnR_Flow]]
→ Verify tại: [[Signoff]]
→ Derived from: Product Requirements (MRD/PRD) — system level, ngoài scope PD vault
→ Closely related: [[TimingClosure]] · [[PowerAnalysis]] · [[IRDrop]] · [[DRC]] · [[Electromigration]]