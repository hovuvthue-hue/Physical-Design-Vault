---
tags: [chain, lef-geometry, pnr-flow]
concepts: [Pitch, Track, RoutingGrid, Site, Row, PlacementGrid, CellAbstract, Pin, Obstruction, LEF, DEF, StandardCell, HardIP, TieCell, TapCell, EndCapCell, DecapCell, FillerCell]
---

# Chain: LEF to PnR

## Mục đích
Thể hiện quan hệ nhân quả có hướng từ các thông số technology cơ bản trong Tech LEF (Pitch, Site) qua các cấu trúc vật lý trung gian (Track, RoutingGrid, Row, PlacementGrid, CellAbstract) đến các inputs của PnR flow, làm rõ tại sao Routing Grid và Placement Grid phải align với nhau.

## Inputs của Chain

[[LEF]] (Tech LEF + Macro LEF) · [[GDS]] (Standard Cell + Hard IP layouts) · [[ITF]] · [[GateLevelNetlist]]

## Hai nhánh song song từ Tech LEF

**Nhánh 1 — Routing Grid:**

Tech LEF LAYER section định nghĩa [[Pitch]] per metal layer.
Pitch instantiates thành các [[Track]]s: $\text{Track}_i = \text{Origin} + i \times \text{Pitch}$.
Tất cả Tracks trên tất cả layers tạo thành [[RoutingGrid]].
RoutingGrid là không gian mà [[Routing]] tool hoạt động.

**Nhánh 2 — Placement Grid:**

Tech LEF SITE section định nghĩa [[Site]] (width = 1 M1 Pitch, height = N × M1 Pitch).
Site tiles theo chiều ngang tạo thành [[Row]]s (stored trong [[DEF]] post-Floorplan).
Tất cả Rows tiling Core area tạo thành [[PlacementGrid]].
PlacementGrid là không gian mà [[Placement]] tool hoạt động.

## KEY CONSTRAINT — Alignment giữa hai nhánh

Routing Grid ← must align → Placement Grid
Site Width   = M1 Pitch    (ensures M1 Tracks align with cell X boundaries)
Site Height  = N × M1 Pitch (ensures M1 Tracks align with Pin Y positions)

Alignment được Foundry đảm bảo khi define Site dimensions dựa trên Pitch values. Nếu không align: mọi Pin access cần jog wire → routing congestion → timing degradation.

## Cell Abstract — nơi hai nhánh hội tụ

[[GDS]] (full layout) → Abstract Generator → [[CellAbstract]] (Macro LEF)

CellAbstract chứa:
- SIZE: kích thước ngoài → dùng bởi PlacementGrid (Placement)
- [[Pin]] locations: phải align với RoutingGrid Track intersections → dùng bởi Routing
- [[Obstruction]] regions: internal metal blocking → Routing phải avoid

## Chuỗi nhân quả đầy đủ

**Từ Tech LEF đến Routing:**

[[Pitch]] định nghĩa Track spacing → [[Track]]s instantiate trên Core area → [[RoutingGrid]] formed → [[Routing]] allocates Tracks cho nets → Router access [[Pin]]s, avoids [[Obstruction]]s

**Từ Tech LEF đến Placement:**

[[Site]] định nghĩa minimum cell unit → [[Row]]s formed bằng tiling Sites → [[PlacementGrid]] = all Site positions → [[Placement]] places [[StandardCell]] instances tại Site-aligned positions

[[StandardCell]] library cũng bao gồm physical-only cells như [[TieCell]], [[TapCell]], [[EndCapCell]], [[DecapCell]], [[FillerCell]], được dùng/insertion trong [[Floorplanning]] / [[Placement]] và được thể hiện ở implementation checkpoints như [[DEF]].

**Hard IP path:**

[[HardIP]] layout → Abstract Generator → [[CellAbstract]] với large [[Obstruction]] → [[Floorplanning]] places Hard IP với FIXED status → Routing channels planned around OBS extent

## Quan hệ giữa các concepts

| From | To | Quan hệ | Ghi chú |
|---|---|---|---|
| Pitch | Track | instantiates | Track spacing = Pitch |
| Track | RoutingGrid | composes | RoutingGrid = all Tracks on all layers |
| Site | Row | tiles into | Row = N Sites horizontal |
| Row | PlacementGrid | composes | PlacementGrid = all Rows on Core area |
| Pitch | Site | constrains | Site Width = M1 Pitch (alignment rule) |
| RoutingGrid | PlacementGrid | must align with | KEY CONSTRAINT |
| GDS | CellAbstract | abstracted into | Abstract Generator strips non-PnR layers |
| CellAbstract | Pin | contains | Pin locations must be on RoutingGrid |
| CellAbstract | Obstruction | contains | OBS = internal metals, forbidden for external routing |
| LEF | Routing | defines rules for | Tech LEF = routing rules, Macro LEF = cell constraints |
| DEF | Placement | stores state for | DEF updated at each PnR checkpoint |

## Điểm giao quan trọng với chains khác

**Chain_PnR_Flow:** [[LEF]] và [[DEF]] là inputs của Import step trong Chain_PnR_Flow; [[RoutingGrid]] và [[PlacementGrid]] là infrastructure được setup trong Floorplanning và consumed bởi tất cả downstream PnR steps; [[CellAbstract]] của [[HardIP]] là critical input cho Floorplanning.

**Chain_STA_Basics:** [[Pin]] locations (từ CellAbstract) ảnh hưởng đến wire length estimates trong pre-route [[STA]]; [[ITF]] Metal Stack parameters → [[ParasiticExtraction]] → [[SPEF]] → Net Delay trong STA.

**Chain_BackEnd_Overview:** LEF Geometry là foundation layer của toàn bộ Back-End Design — trước khi bất kỳ PnR step nào có thể chạy, tất cả LEF files phải loaded và validated.

## Concepts trong chain này

[[Pitch]] · [[Track]] · [[RoutingGrid]] · [[Site]] · [[Row]] · [[PlacementGrid]] · [[CellAbstract]] · [[Pin]] · [[Obstruction]] · [[LEF]] · [[DEF]] · [[StandardCell]] · [[TieCell]] · [[TapCell]] · [[EndCapCell]] · [[DecapCell]] · [[FillerCell]] · [[HardIP]] · [[MetalStack]] · [[ITF]]