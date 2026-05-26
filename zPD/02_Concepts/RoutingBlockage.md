---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Floorplanning step — created as part of Macro keep-out strategy
used_by: [Routing, MacroPlacement, Floorplanning]
requires: [CoreArea, RoutingGrid, MacroPlacement]
chain: Chain_PnR_Flow
---
# RoutingBlockage

## Definition
Routing Blockage là vùng trong Core Area được đánh dấu cấm Router đặt wires trên một hoặc nhiều metal layers cụ thể. Khác với Placement Blockage (cấm Standard Cells), Routing Blockage không ảnh hưởng đến Placement — Standard Cells vẫn có thể được placed bên trong vùng Routing Blockage, nhưng Router không được sử dụng các layers bị block trong vùng đó.

**Phân biệt Routing Blockage vs Placement Blockage:**

| | Routing Blockage | Placement Blockage |
|---|---|---|
| Cấm gì | Wire placement trên specified layers | Standard Cell / Macro placement |
| Placement OK? | Có | Không (theo loại) |
| Routing OK? | Không (trên blocked layers) | Có |
| Scope | Layer-specific | Toàn bộ cell stack |

**Mối quan hệ với Macro Obstruction:**

Macro OBS trong CellAbstract LEF tự động block routing bên trong Macro body. Routing Blockage là một công cụ bổ sung, được PD engineer đặt thủ công ngoài phạm vi Macro body để mở rộng vùng cấm routing ra các khu vực lân cận.

## Computed from

Routing Blockages được tạo thủ công tại những vùng có routing sensitivity cao:

**Vùng quanh analog/sensitive Macros:** Block các digital signal layers để tránh digital switching noise coupling vào analog circuits. Ví dụ: block M2–M4 trong vùng quanh PLL hoặc ADC.

**Vùng power strap corridors:** Block signal routing layers trong vùng reserved cho power straps để đảm bảo Router không sử dụng chúng cho signal wires.

**Vùng clock spine:** Block non-clock layers trong vùng dedicated cho clock distribution để tránh SI (Signal Integrity) coupling với clock nets.

Congestion có thể được giải quyết bằng cả Placement Blockage (giảm density của Standard Cells) lẫn Routing Blockage (redirect routing traffic sang layers khác). Cả hai là complementary tools.

## Constrains
- **[[Routing]]**: Router không được đặt wires trên blocked layers trong vùng Routing Blockage; phải route qua các layers không bị block hoặc đi vòng qua vùng khác

## Requires
- [[CoreArea]] — Routing Blockages được đặt bên trong Core Area
- [[RoutingGrid]] — Blockage effective area được tính theo Track grid của các layers bị block

## Used by
- [[Routing]] — Router read Routing Blockages khi xây dựng routing graph
- [[Floorplanning]] — Routing Blockages là output của Floorplanning step
- [[MacroPlacement]] — Routing Blockages thường đi kèm với Macro Halos

## Key insight
[USER REVIEW — draft suggestion]: Routing Blockage là công cụ tinh tế hơn Placement Blockage vì nó cho phép Standard Cells vẫn được placed trong vùng đó (để duy trì Utilization) trong khi vẫn redirect routing traffic. Use case điển hình: đặt Routing Blockage trên M3 và M4 trong chimney giữa hai Macros, trong khi đặt Soft Placement Blockage trong cùng vùng — Standard Cells và Buffers vẫn được placed nhưng routing phải đi qua M2 và M5 thay vì gây congestion trên middle layers.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Complements: [[PlacementBlockage]]
→ Scope: Layer-specific (specified layers only)
→ vs. [[Obstruction]] in CellAbstract: OBS là internal cell property; Routing Blockage là external design-level constraint
→ Consumed by: [[Routing]] · [[Floorplanning]]
→ Cùng nhóm: [[PlacementBlockage]] · [[MacroPlacement]] · [[RoutingGrid]]