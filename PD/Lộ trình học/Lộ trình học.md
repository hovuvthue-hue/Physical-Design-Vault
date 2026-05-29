# Physical Design Learning Roadmap

> Mục tiêu của file này không phải là liệt kê concept card theo thứ tự học.  
> Mục tiêu là xây dựng một bài giảng tuyến tính về Physical Design cho người mới, trong đó các concept card được gọi bằng wikilink đúng lúc khái niệm đó xuất hiện.

## Bố cục bài giảng

### Chương 1 — Inputs: trước khi chạy Physical Design phải có gì?

Dạy người mới rằng tool không “tự hiểu chip”. Nó cần nhiều lớp mô tả khác nhau:

```markdown
## 1. Inputs của Physical Design

Trước khi làm [[Floorplanning]], tool cần load các artifact chính:

- [[GateLevelNetlist]]: cho biết design gồm những cell nào và connectivity ra sao.
- [[SDC]]: cho biết clock, timing constraint, input/output delay, false path, multicycle path.
- [[LIB]]: timing/power model của standard cell.
- [[LEF]]: abstract physical view của cell/macro và technology information.
- [[DEF]]: carrier của physical state, có thể chứa floorplan, placement, routing state.
- [[MMMC]]: framework phân tích nhiều mode và nhiều corner.
- [[PhysicalConstraints]]: boundary, pin location, macro constraint, die/core rule.

Ở giai đoạn này, mục tiêu chưa phải tối ưu PPA. Mục tiêu là xác nhận input đủ sạch để đi tiếp. Nếu [[DesignImport]] fail, lỗi thường nằm ở synthesis, constraint hoặc library setup; không nên cố chạy tiếp sang floorplan.
```

`Chain_PnR_Flow` ghi rõ Design Import nhận `GateLevelNetlist`, `LEF`, `MMMC`, `PhysicalConstraints`, chạy các check ban đầu, và exit criteria gồm không black box, không error, timing coverage đủ, WNS/TNS sạch ở zero-RC nếu flow yêu cầu.

### Chương 2 — Geometry foundation: vì sao phải hiểu LEF trước khi hiểu placement/routing?

Đây là chỗ dùng các card như `[[Pitch]]`, `[[Track]]`, `[[Site]]`, `[[Row]]`, `[[PlacementGrid]]`, `[[RoutingGrid]]`.

```markdown
## 2. Geometry foundation: placement grid và routing grid

Trước khi học [[Placement]], phải hiểu design được đặt và route trên hai loại “lưới” khác nhau:

- [[PlacementGrid]]: nơi standard cells được đặt.
- [[RoutingGrid]]: nơi wires được route.

Hai grid này không độc lập. [[LEF]] định nghĩa [[Pitch]] của routing layers, từ đó sinh ra [[Track]] và [[RoutingGrid]]. Đồng thời [[LEF]] định nghĩa [[Site]], từ đó floorplan tạo [[Row]] và [[PlacementGrid]].

Nếu placement grid và routing grid không align, pin access trở nên khó, router phải tạo jog/detour, congestion tăng và timing xấu đi.

Xem thêm: [[Chain_LEF_to_PnR]].
```

Đây là một chương rất quan trọng cho người mới, vì nó giải thích tại sao `Site`, `Row`, `Track`, `Pitch` không phải kiến thức rời rạc. `Chain_LEF_to_PnR` đã mô tả trực tiếp: Tech LEF `LAYER` định nghĩa `Pitch`, tạo `Track` và `RoutingGrid`; Tech LEF `SITE` định nghĩa `Site`, từ đó tạo `Row` và `PlacementGrid`; hai nhánh phải align.

### Chương 3 — Floorplanning: tạo “không gian vật lý” cho chip

```markdown
## 3. Floorplanning: tạo search space cho toàn bộ flow

[[Floorplanning]] là bước đầu tiên biến logical design thành physical plan. Nó quyết định:

- [[CoreArea]] và boundary.
- I/O hoặc block pin planning.
- [[MacroPlacement]].
- [[PlacementBlockage]] và [[RoutingBlockage]].
- [[Row]] và [[PlacementGrid]].
- [[PDN]].
- early routability, early timing và early [[IRDrop]] feasibility.

Một câu cần nhớ:

> Floorplanning không tối ưu từng cell; floorplanning định nghĩa không gian mà mọi tối ưu sau đó bị giới hạn bên trong.

Nếu floorplan sai, [[Placement]], [[ClockTreeSynthesis]], [[Routing]] và [[Signoff]] có thể đều phải trả giá.
```

Trong repo, Floorplanning nhận validated design DB, định nghĩa boundary, IO, macro placement, standard-cell area, PDN, blockages, rows, routing grid, rồi lưu dưới dạng DEF/checkpoint; output của nó trở thành hard constraint cho placement. Reference floorplanning cũng nhấn mạnh floorplan gồm boundary definition, I/O planning, macro placement, legal standard-cell region, PDN và quality checks, không phải một bước đơn lẻ.

### Chương 4 — Placement: không chỉ là “đặt cell”

```markdown
## 4. Placement: tối ưu standard cells trước CTS

[[Placement]] nhận floorplan đã được fix: core boundary, macro positions, rows, blockages, PDN và placement grid. Nhiệm vụ chính là đặt [[StandardCell]] vào legal site/row positions, nhưng ở mức học nghiêm túc, placement bao gồm nhiều lớp:

1. [[GlobalPlacement]]: tìm vị trí gần đúng, có thể còn overlap.
2. [[Legalization]] / [[DetailedPlacement]]: snap cell vào legal [[Site]] và [[Row]].
3. Timing-driven optimization: cải thiện setup trước CTS.
4. [[DRVFixing]]: sửa max slew, max cap, max fanout.
5. [[CongestionAnalysis]]: kiểm tra khả năng route.
6. [[PowerOptimization]], [[SpareCell]], [[MBFF]], tie-cell handling nếu flow có.

Một câu cần nhớ:

> Floorplanning shapes the legal search space; Placement optimizes within that space; CTS and Routing inherit Placement’s consequences.
```

Reference placement trong repo đã nói rõ Placement là một “pre-CTS implementation and optimization package”, không chỉ là thao tác đặt standard cells lên row. Nó bao gồm global placement, detailed placement, DRV fixing, pre-CTS timing optimization, power/area optimization, spare cells, MBFF và congestion analysis.

### Chương 5 — STA đi xuyên suốt Physical Design

```markdown
## 5. STA: hệ thống đo pass/fail của Physical Design

[[STA]] không phải một bước riêng cuối flow. Nó chạy nhiều lần, với độ chính xác tăng dần:

- Post-Import STA: zero-RC hoặc estimate rất thô.
- Post-Placement STA: ideal clock, estimated wire delay.
- Post-CTS STA: propagated clock, real [[ClockSkew]] và [[ClockLatency]].
- Post-Route STA: actual extracted RC từ [[SPEF]].
- Signoff STA: full [[MMMC]] across modes/corners.

Các khái niệm phải học ở đây:

[[CellDelay]], [[NetDelay]], [[StageDelay]], [[Slew]], [[TimingArc]], [[Slack]], [[SetupTime]], [[HoldTime]], [[ClockUncertainty]], [[ClockSkew]], [[ClockLatency]], [[PropagationDelay]], [[MMMC]], [[SDC]].
```

`Chain_STA_Basics` đã dựng đúng chuỗi từ `LIB` → `CellDelay` → `NetDelay` → `StageDelay` → `STA` → `Slack` → phán định setup/hold. Nó cũng chỉ rõ STA chạy tại nhiều checkpoint với độ chính xác tăng dần: Post-Import, Post-Placement, Post-CTS, Post-Route, Signoff.

### Chương 6 — CTS: biến ideal clock thành real clock

```markdown
## 6. Clock Tree Synthesis: từ ideal clock sang physical clock network

Trước [[ClockTreeSynthesis]], clock thường được timing như ideal clock. Sau CTS, clock có buffer, inverter, wire, RC, skew và insertion delay thật.

CTS cần [[Placement]] trước, vì nó cần biết tọa độ của clock sinks. Output của CTS gồm clock-tree topology, inserted clock buffers/inverters, clock net structure và timing reports.

Các khái niệm phải học:

[[ClockTreeSynthesis]], [[CTSFlow]], [[CTSOptimization]], [[CTSQualityReview]], [[ClockSkew]], [[ClockLatency]], [[Slew]], [[HoldTime]], [[PostCTSOptimization]].

Điểm quan trọng: hold violation thường trở nên rõ hơn sau CTS, vì propagated clock thay thế ideal clock.
```

Trong `Chain_PnR_Flow`, CTS nhận clock sink locations từ Placement, tạo clock tree topology, buffer/inverter positions và clock net routes; output của CTS trở thành fixed constraint cho Routing. Reference CTS của repo cũng mô tả CTS chạy sau Placement, trước Routing, cần placed design để biết clock sink locations, và mục tiêu chính là skew, insertion delay, slew/transition và clock power.

### Chương 7 — Routing: biến connectivity thành wire geometry

```markdown
## 7. Routing: từ net connectivity sang wire geometry

[[Routing]] nhận placement đã legal, clock tree đã tạo, PDN đã có, routing grid đã định nghĩa. Nhiệm vụ của routing là tạo wire geometries cho signal nets và clock nets.

Người mới nên học routing theo hai lớp:

1. Global routing: tìm đường đi xấp xỉ, đánh giá congestion, tạo routing guide.
2. Detailed routing: gán track/via cụ thể và cố gắng đạt DRC-clean.

Các khái niệm cần gọi bằng wikilink:

[[Routing]], [[RoutingGrid]], [[Track]], [[Pitch]], [[Pin]], [[Obstruction]], [[RoutingBlockage]], [[CongestionAnalysis]], [[MetalStack]], [[DRC]], [[ParasiticExtraction]].
```

`Chain_PnR_Flow` mô tả Routing nhận clock tree topology từ CTS và PDN từ floorplanning, output là wire geometries hoàn chỉnh cho signal nets, sau đó wire geometry được truyền sang Parasitic Extraction.

### Chương 8 — Parasitic Extraction và Signoff: từ hình học về timing thật

```markdown
## 8. Extraction and Signoff: kiểm chứng bằng dữ liệu vật lý thật

Sau routing, wire không còn là ước lượng. Tool đã có hình học thật: length, width, spacing, via, layer, coupling. [[ParasiticExtraction]] dùng hình học đó và technology/extraction rules để tạo [[SPEF]].

[[SPEF]] đưa RC thật vào [[STA]], làm [[NetDelay]] chính xác hơn. Đây là lý do Post-Route STA và Signoff STA đáng tin hơn Post-Placement STA.

[[Signoff]] kiểm tra:

- timing across [[MMMC]];
- DRC;
- LVS;
- IR drop / EM;
- physical verification;
- final GDS readiness.

Nếu fail, flow phải ECO lại từ bước bị ảnh hưởng.
```

Trong repo, Parasitic Extraction nhận wire geometries từ Routing và xuất `SPEF`, còn Signoff nhận SPEF, GDS và netlist để ra quyết định GDS release. `Chain_PnR_Flow` cũng đã có ECO loop: setup violation quay lại Placement/CTS, hold quay lại CTS, DRC quay lại Routing, IR drop hoặc congestion nặng có thể quay lại Floorplanning.

## Các card nên được gọi đúng thời điểm

Không nên gom card theo folder. Nên gọi theo ngữ cảnh:

|Khi bài giảng nói về|Wikilink nên xuất hiện|
|---|---|
|Back-end boundary|`[[LogicSynthesis]]`, `[[GateLevelNetlist]]`, `[[Physical Design]]`, `[[Signoff]]`|
|Input setup|`[[SDC]]`, `[[LIB]]`, `[[LEF]]`, `[[DEF]]`, `[[MMMC]]`, `[[PhysicalConstraints]]`|
|Geometry foundation|`[[Pitch]]`, `[[Track]]`, `[[RoutingGrid]]`, `[[Site]]`, `[[Row]]`, `[[PlacementGrid]]`|
|Library abstraction|`[[CellAbstract]]`, `[[Pin]]`, `[[Obstruction]]`, `[[StandardCell]]`, `[[HardIP]]`|
|Floorplanning|`[[Floorplanning]]`, `[[CoreArea]]`, `[[MacroPlacement]]`, `[[PDN]]`, `[[IRDrop]]`, `[[PlacementBlockage]]`, `[[RoutingBlockage]]`|
|Placement|`[[Placement]]`, `[[GlobalPlacement]]`, `[[DetailedPlacement]]`, `[[Legalization]]`, `[[PlacementDensity]]`, `[[CongestionAnalysis]]`, `[[DRVFixing]]`|
|CTS|`[[ClockTreeSynthesis]]`, `[[CTSFlow]]`, `[[ClockSkew]]`, `[[ClockLatency]]`, `[[Slew]]`, `[[PostCTSOptimization]]`|
|Timing|`[[STA]]`, `[[CellDelay]]`, `[[NetDelay]]`, `[[StageDelay]]`, `[[Slack]]`, `[[SetupTime]]`, `[[HoldTime]]`|
|Routing/extraction|`[[Routing]]`, `[[ParasiticExtraction]]`, `[[SPEF]]`, `[[GDS]]`|
|Closure|`[[Signoff]]`, `[[DRC]]`, `[[LVS]]`, `[[IRDrop]]`, `[[EM]]`, `[[ECO]]`|

Một số card như `[[DRC]]`, `[[LVS]]`, `[[EM]]`, `[[ECO]]`, `[[Halo]]`, `[[BlockIOPin]]`, `[[PadRing]]`, `[[PowerStripe]]`, `[[CoreRing]]` có thể chưa tồn tại hoặc chưa thấy rõ trong search. Reference floorplanning của bạn cũng đã đề xuất nhiều new-card candidates, đặc biệt cho die/core area, IO/pad, blockage/halo, PDN, IR drop, EM, ESD, seal ring và packaging-aware top-level work.
