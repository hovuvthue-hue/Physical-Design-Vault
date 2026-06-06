## 4.1. Mục tiêu của chương

Sau chương này, người học cần hiểu:

- [[Floorplanning]] là bước major đầu tiên sau [[DesignImport]].
- Floorplanning không chỉ là chọn kích thước chip hoặc core.
- Floorplanning tạo ra physical search space cho toàn bộ flow phía sau.
- Các quyết định ở Floorplanning trở thành hard constraints cho [[Placement]], [[ClockTreeSynthesis]], [[Routing]], [[PowerAnalysis]] và [[Signoff]].
- Một floorplan xấu thường không thể được “cứu” hoàn toàn bằng Placement hoặc Routing optimization.
- Floorplanning phải cân bằng nhiều mục tiêu cùng lúc: area, timing, power, routability, macro integration, IO/pin planning, PDN và signoff feasibility.

Một câu cần nhớ:

> Floorplanning định nghĩa không gian vật lý mà mọi tối ưu sau đó bị giới hạn bên trong.

---

## 4.2. Floorplanning nằm ở đâu trong PnR flow?

Luồng cơ bản:

```text
[[DesignImport]]
  ↓
[[Floorplanning]]
  ↓
[[Placement]]
  ↓
[[ClockTreeSynthesis]]
  ↓
[[Routing]]
  ↓
[[ParasiticExtraction]]
  ↓
[[Signoff]]
```

[[Floorplanning]] nhận **validated design database** từ [[DesignImport]].

Điều đó có nghĩa là trước khi floorplan, các điều kiện cơ bản phải đã pass:

- [[GateLevelNetlist]] đọc được;
- [[LEF]] / [[CellAbstract]] đầy đủ;
- [[LIB]] và [[MMMC]] hợp lệ;
- [[SDC]] resolve đúng;
- không có [[Black Box]] ngoài ý muốn;
- zero-RC timing sanity check đủ sạch;
- timing coverage đủ tốt;
- [[PhysicalConstraints]] đã load;
- power/ground net naming có context hợp lệ.

Nếu Design Import chưa sạch, Floorplanning sẽ xây trên nền dữ liệu sai.

---

## 4.3. Floorplanning giải bài toán gì?

Floorplanning là bước chuyển từ:
```text
Logical design database
```
sang:
```text
Physical implementation plan
```
Nó quyết định các yếu tố vĩ mô của layout:

- [[CoreArea]];
- die/core boundary;
- aspect ratio;
- target utilization;
- full-chip IO pad placement hoặc block-level pin placement;
- [[MacroPlacement]];
- [[PlacementBlockage]];
- [[RoutingBlockage]];
- [[Row]] / [[PlacementGrid]];
- [[RoutingGrid]] setup;
- [[PDN]];
- physical-only cell strategy;
- early congestion;
- early timing feasibility;
- early [[IRDrop]] feasibility.

Không nên hiểu Floorplanning là bước “vẽ khung chip”. Nó là bước đặt ra toàn bộ điều kiện ban đầu cho implementation.

Một cách nhìn đúng hơn:

> Floorplanning là architectural planning ở physical level.

---

## 4.4. Inputs của Floorplanning

[[Floorplanning]] cần các nhóm input chính sau:

```text
Validated design database từ [[DesignImport]]
[[GateLevelNetlist]]
[[SDC]]
[[MMMC]]
[[LEF]]
[[LIB]]
[[DEF]] nếu có initial physical state
[[PhysicalConstraints]]
[[CoreArea]] estimate
[[IOCell]] hoặc block pins
[[HardIP]]
Power/Ground net context
```

Vai trò từng nhóm:

- [[GateLevelNetlist]] cho biết cell count, hierarchy, macro instances và connectivity.
- [[SDC]] cho biết timing intent, critical clocks và timing paths quan trọng.
- [[MMMC]] cho biết mode/corner analysis context.
- [[LEF]] cho biết cell/macro size, pins, obstructions, sites, tracks và routing layers.
- [[LIB]] cho timing/power model.
- [[PhysicalConstraints]] cho die/core/boundary/pin/IO constraints.
- [[HardIP]] cung cấp macro footprint, pin locations, OBS và timing models.
- [[IOCell]] cần cho full-chip pad ring/IO planning.
- Power/Ground net context cần cho [[PDN]] setup.

Một floorplan tốt không thể được tạo chỉ từ area number. Nó cần connectivity, timing intent, physical abstracts, power context và interface constraints.

---

## 4.5. Outputs của Floorplanning

Output của Floorplanning thường là một physical database/checkpoint và có thể được lưu dưới dạng [[DEF]] hoặc database nội bộ của tool.

Các output chính:

```text
Core boundary
Die/core relationship
Aspect ratio
Target utilization
IO pad positions hoặc block pin locations
Macro FIXED positions
Macro halos / keep-outs
Placement blockages
Routing blockages
Row definitions
PlacementGrid
RoutingGrid setup
PDN rings / straps / rails context
Initial physical-only cell strategy
Floorplan quality report
```

Quan trọng nhất: output của Floorplanning trở thành input cứng cho Placement.

```text
[[Floorplanning]]
  ↓ hard constraints
[[Placement]]
```

Cụ thể, [[Placement]] nhận:

- core boundary;
- macro fixed locations;
- row definitions;
- placement blockages;
- standard-cell legal regions;
- utilization constraints.

Nếu các thứ này sai, Placement có thể vẫn chạy, nhưng QoR downstream sẽ xấu.

---

## 4.6. Hai cấp độ Floorplanning: full-chip và block-level

Floorplanning có hai ngữ cảnh lớn.

### Full-chip Floorplanning

Ở full-chip, floorplan phải xử lý:

- die size;
- core area;
- IO ring;
- IO pad placement;
- corner cells;
- power supply cells;
- ESD context;
- package constraints;
- seal ring / scribe line constraints;
- top-level macro placement;
- chip-level PDN.

Không nên nói “PD engineer tự do quyết định die size”. Cách hiểu đúng hơn:

> Die size là kết quả của ràng buộc system, package, pad count, cost và physical feasibility. PD engineer/floorplan team tham gia trade-off, nhưng không chọn tùy ý.

Các yếu tố full-chip quan trọng:

- pad-limited hay core-limited;
- package type;
- bond-wire hoặc flip-chip;
- IO count;
- IO voltage domains;
- core-to-IO spacing;
- ESD strategy;
- power delivery từ package vào die.

### Block-level Floorplanning

Ở block-level, designer thường nhận nhiều ràng buộc từ top-level:

- PR boundary;
- block shape;
- block pin locations;
- pin layer;
- pin side;
- macro constraints;
- top-level integration assumptions.

Block designer thường **không tự do đổi boundary hoặc pin locations** nếu chưa đồng bộ với top-level.

Điểm quan trọng:

> Ở block-level, pin/boundary là interface contract với top-level. Sửa tùy tiện có thể phá integration.

---

## 4.7. Sáu mục tiêu của Floorplanning

Một floorplan tốt phải đồng thời phục vụ nhiều mục tiêu. Không có mục tiêu nào đứng riêng.

### 1. Define physical boundaries

Gồm:

- die size;
- core size;
- aspect ratio;
- core-to-IO spacing;
- PR boundary nếu là block-level;
- legal region cho placement/routing.

Boundary không chỉ là hình chữ nhật. Ở block-level, boundary có thể rectilinear như L-shape hoặc T-shape để fit vào chip topology.

### 2. Optimize PPA ở mức vĩ mô

Floorplanning ảnh hưởng trực tiếp đến:

- Performance: wire length, macro distance, clock distribution.
- Power: switching capacitance, PDN quality, macro clustering.
- Area: die/core utilization, whitespace, macro packing.
- Routability: routing channel, congestion, pin access.
- Manufacturability: grid alignment, boundary, DRC risk.

Ở mức này, PPA không được tối ưu bằng cell sizing chi tiết. Nó được tối ưu bằng topology vật lý.

### 3. Enable clean IO / pin planning

Full-chip cần đặt [[IOCell]] / IO pads đúng vị trí.

Block-level cần đặt block pins đúng boundary/interface expectation.

IO/pin planning ảnh hưởng:

- timing từ IO đến logic;
- congestion tại boundary;
- top-level integration;
- package connectivity;
- routing channel từ IO ring vào core;
- ESD và power domain structure ở full-chip.

### 4. Place macros and define standard-cell areas

[[MacroPlacement]] là một trong các quyết định floorplan quan trọng nhất.

Sau khi đặt macros, phần còn lại của core mới trở thành vùng đặt standard cells.

Floorplan phải tránh:

- standard-cell area bị chia nhỏ;
- chimney quá hẹp giữa macros;
- notch formation;
- macro pins quay vào hướng khó route;
- macros chặn routing channels chính;
- macros đặt quá xa logic/interface liên quan;
- macros đặt xa clock/power resources quan trọng.

### 5. Build robust [[PDN]]

[[PDN]] phải đưa VDD/VSS từ nguồn cấp vào toàn bộ core, standard cells và macros.

PDN cần cân bằng:

- đủ robust để giảm [[IRDrop]];
- đủ current capacity để giảm EM risk;
- không quá dày đến mức chiếm hết routing resources;
- kết nối được tới standard-cell rails;
- kết nối được tới macro PG pins;
- phù hợp với metal stack.

Một PDN yếu có thể gây timing degradation hoặc signoff failure.

### 6. Prepare critical pre-routes nếu cần

Một số design có thể cần pre-route hoặc route planning cho:

- clock/reset;
- analog-sensitive signals;
- high-speed interfaces;
- power-critical nets;
- shielding-related structures.

Không phải flow nào cũng làm nhiều pre-route ở Floorplanning. Nhưng về mặt khái niệm, Floorplanning phải chừa đủ không gian và tài nguyên routing cho các nets nhạy cảm.

---

## 4.8. Bước 1 — Define Boundary

Bước đầu tiên là xác định boundary và [[CoreArea]].

Các tham số chính:

- die area;
- core area;
- aspect ratio;
- target utilization;
- core-to-IO spacing;
- margin với pad ring hoặc block boundary;
- site/grid snapping;
- row feasibility.

### Core Area

[[CoreArea]] là vùng chứa:

- [[StandardCell]];
- [[HardIP]] / macros;
- placement rows;
- PDN inside core;
- routing resources cho core logic.

Một công thức khái niệm:

```text
Core Area ≈ Estimated Physical Area / Target Utilization
```

Trong đó estimated physical area có thể gồm:

- total standard-cell area;
- macro area;
- expected CTS/timing buffer overhead;
- spare/ECO overhead nếu có;
- whitespace cần cho routing;
- whitespace cần cho optimization.

Không nên dùng công thức này như luật cứng. Đây là estimation ban đầu, sau đó phải được kiểm chứng bằng congestion, timing và power checks.

### Target Utilization

Target utilization quá cao:

- cell density cao;
- routing congestion tăng;
- pin access khó;
- buffer insertion thiếu chỗ;
- timing closure khó;
- detailed routing có thể fail.

Target utilization quá thấp:

- die/core lớn;
- wire length có thể tăng;
- clock tree dài hơn;
- area cost cao;
- power có thể tăng do wire capacitance lớn.

Vì vậy utilization là trade-off, không phải càng cao càng tốt.

### Core Utilization vs Standard Cell Utilization

Cần phân biệt:

```text
Core Utilization =
(Standard Cell Area + Macro Area) / Total Core Area
```

và:

```text
Standard Cell Utilization =
Total Standard Cell Area / Placeable Standard Cell Area
```

Metric thứ hai thường sát với placement difficulty hơn, vì macros, blockages, halos và unusable regions làm giảm placeable area thật.

### Aspect Ratio

Trong vault này, nên thống nhất:

```text
Aspect Ratio = Width / Height
```

```text
AR = 1.0  → square
AR > 1.0 → wide
AR < 1.0 → tall
```

Cảnh báo:

> Một số tool hoặc tài liệu dùng định nghĩa ngược lại, ví dụ Height / Width. Không được so sánh aspect ratio nếu chưa biết convention.

Aspect ratio ảnh hưởng:

- chiều dài các đường data path chính;
- số lượng routing resources theo từng hướng;
- clock tree topology;
- macro arrangement;
- khả năng fit với package/top-level.

---

## 4.9. Bước 2 — Place IOs hoặc assign block pins

Bước này phụ thuộc cấp độ design.

### Full-chip IO placement

Ở full-chip, cần đặt [[IOCell]] hoặc IO pads quanh die.

Các yếu tố cần xét:

- package pinout;
- wire bond / bump constraints;
- no-crossing rule;
- max bond-wire length;
- bond-wire angle;
- ESD structure;
- IO power ring;
- core power ring;
- IO voltage domains;
- analog/digital separation;
- interface grouping.

IO cells không giống standard cells thông thường. Chúng lớn hơn, chịu tải ngoài chip lớn hơn và thường chứa ESD/pad-related structures.

### Block-level pin assignment

Ở block-level, thay vì IO pads, ta có block pins.

Block pin planning cần xét:

- top-level connectivity;
- related registers hoặc macros bên trong block;
- congestion tại boundary;
- pin layer;
- pin spacing;
- timing-critical interfaces;
- distribution đều trên boundary;
- tránh cluster quá dày.

Một block pin đặt sai có thể làm top-level routing detour hoặc làm block internal routing congested.

### Hiểu lầm cần tránh

Sai:

> IO/pin placement là chuyện phụ, để routing xử lý sau.

Đúng:

> IO/pin placement là physical interface contract. Nếu interface sai, routing và timing sẽ trả giá ở nhiều cấp độ.

---

## 4.10. Bước 3 — Macro Placement

[[MacroPlacement]] là quá trình đặt [[HardIP]] / macros vào [[CoreArea]].

Macros có thể là:

- SRAM;
- ROM;
- register file;
- PLL;
- ADC/DAC;
- PHY;
- SERDES;
- analog/mixed-signal IP;
- large predesigned blocks.

Macros khác standard cells vì:

- kích thước lớn;
- shape cố định;
- pin locations cố định;
- internal routing bị che bởi [[Obstruction]];
- cần LEF/LIB/GDS/Verilog/CDL views;
- thường phải FIXED trước Placement;
- ảnh hưởng mạnh tới routing channels, PDN, timing và clock.

### Vì sao Macro Placement khó?

Vì nó là bài toán multi-objective.

Một macro placement tốt phải xét ít nhất 7 nhóm yếu tố:

1. Timing closure.
2. Power consumption.
3. System interface.
4. Clock distribution.
5. Routing congestion.
6. Power delivery / EM / thermal.
7. Placement rules / foundry requirements.

Không nên chỉ đặt macros “gần nhau cho ngắn dây”. Đó chỉ là một phần.

---

## 4.11. Flight-line analysis

Flight-line analysis là cách nhìn connectivity giữa:

- macro ↔ macro;
- macro ↔ IO/pins;
- macro ↔ standard-cell logic;
- macro ↔ clock/power-related regions.

Mục tiêu:

- giảm tổng wire length;
- giảm crisscross connections;
- đặt macros theo data flow;
- phát hiện macro nào nên gần nhau;
- phát hiện macro nào nên gần IO;
- phát hiện vùng có nguy cơ congestion.

Nếu flight lines chéo nhau quá nhiều, layout thường dẫn đến routing detour.

Một nguyên tắc thực tế:

> Trace the hot path first.

Tức là xem critical connectivity/timing path trước, đặt các macro liên quan vào topology hợp lý, sau đó mới xử lý các macro ít critical hơn.

---

## 4.12. Macro orientation

Một macro không chỉ có vị trí. Nó còn có orientation.

Orientation ảnh hưởng:

- pin direction;
- pin accessibility;
- routing channel;
- power connection;
- clock access;
- foundry/legal orientation rules;
- uniform poly orientation nếu process yêu cầu.

Một số nguyên tắc:

- Macros giao tiếp nhiều với nhau → hướng pins về phía nhau nếu hợp lệ.
- Macro giao tiếp với IO → hướng pins về phía IO liên quan.
- Macro giao tiếp với standard-cell area → hướng pins ra vùng logic liên quan.
- Tránh để nhiều important pins nằm sát corner khó route.
- Không chọn orientation vi phạm library/foundry rule.

Macro orientation sai có thể tạo congestion dù vị trí macro nhìn có vẻ hợp lý.

---

## 4.13. Standard-cell area phải liên tục

Sau khi đặt macros, phần còn lại là vùng cho standard cells.

Một floorplan tốt cần tạo standard-cell areas:

- đủ rộng;
- đủ liên tục;
- không bị chia cắt vô lý;
- có routing channels rõ;
- có whitespace hợp lý;
- có access tới PDN;
- có khả năng chèn buffers/timing fixes.

Cần tránh:

- chimney quá hẹp giữa hai macros;
- notch formation ở góc macro;
- vùng standard cell bị chia nhỏ thành nhiều đảo;
- vùng sát macro pin bị nhồi cells quá dày;
- core corner bị congestion;
- macro cluster làm mất routing channel chính.

Một câu cần nhớ:

> Placement cần vùng để tối ưu. Nếu floorplan chia nát standard-cell area, Placement không còn không gian để sửa timing/congestion.

---

## 4.14. Halo và Placement Blockage

[[PlacementBlockage]] là vùng cấm hoặc hạn chế placement.

Nó được dùng để điều chỉnh local cell density và bảo vệ routing resources.

Các loại chính:

- hard blockage;
- partial blockage;
- soft blockage;
- macro-only blockage;
- halo quanh macro.

### Halo

Halo là keep-out region quanh macro.

Mục tiêu:

- chừa routing space quanh macro pins;
- tránh standard cells lấp sát macro edge;
- giữ chỗ cho power tapping;
- giảm pin-access congestion;
- giảm local routing hotspot.

Halo thường đi cùng macro. Khi macro chưa fixed, halo di chuyển cùng macro. Sau khi macro fixed, halo mới ổn định cho downstream optimization.

### Placement Blockage không phải Routing Blockage

[[PlacementBlockage]] điều khiển nơi cells được đặt.

Nó **không** trực tiếp cấm router dùng metal layer.

Nếu muốn cấm hoặc hạn chế routing trên layer cụ thể, đó là [[RoutingBlockage]].

---

## 4.15. Routing Blockage

[[RoutingBlockage]] là vùng cấm routing trên một hoặc nhiều metal layers cụ thể.

Khác với [[PlacementBlockage]], standard cells vẫn có thể được placed trong vùng routing blockage, nhưng router không được dùng các layers bị block trong vùng đó.

Dùng routing blockage khi cần:

- bảo vệ vùng analog/sensitive macros;
- reserve routing resource cho power straps;
- reserve clock-related routing region;
- redirect routing traffic;
- tránh digital switching noise coupling;
- quản lý layer-specific congestion.

Ví dụ khái niệm:

```text
Vùng giữa hai macros:
- dùng soft PlacementBlockage để không nhồi quá nhiều logic;
- dùng RoutingBlockage trên một số layers để redirect routing traffic;
- vẫn cho phép một số buffers/inverters nếu cần timing fix.
```

Cần nhớ:

> Placement Blockage giảm demand. Routing Blockage giảm supply. Hai thứ bổ sung nhau nhưng không thay thế nhau.

---

## 4.16. Bước 4 — Define Standard Cell Areas và physical-only cells

Sau boundary, IO/pins và macro placement, floorplan phải xác định vùng standard-cell placement.

Bước này liên quan đến:

- [[Row]];
- [[PlacementGrid]];
- legal standard-cell regions;
- macro halos;
- placement blockages;
- row cutting around macros;
- physical-only cells.

Các physical-only cells quan trọng:

- [[TapCell]];
- [[EndCapCell]];
- [[DecapCell]];
- [[FillerCell]];
- [[TieCell]] trong một số context.

Không nên trộn vai trò của chúng.

---

## 4.17. Tap Cell

[[TapCell]] là physical-only standard cell dùng để nối well/substrate về bias rails phù hợp.

Mục tiêu:

- duy trì well/substrate bias;
- giảm rủi ro latch-up;
- đảm bảo điều kiện vật lý của standard-cell fabric.

[[TapCell]] không xử lý logic data.

Điểm cần nhớ:

> TapCell phục vụ body-bias / well-substrate integrity, không phải tạo logic constant. Logic constant thuộc [[TieCell]].

Tap insertion rule phụ thuộc PDK/foundry/library. Không nên dùng một spacing number chung cho mọi process.

---

## 4.18. EndCap Cell

[[EndCapCell]] là physical-only cell đặt ở đầu/cuối [[Row]].

Mục tiêu:

- tạo termination hợp lệ cho row edge;
- bảo vệ row boundary;
- duy trì convention hình học/electrical của standard-cell row;
- hỗ trợ downstream signoff legality.

[[EndCapCell]] không thay vai trò của [[TapCell]], [[FillerCell]] hoặc [[DecapCell]].

Một cách hiểu ngắn:

> EndCapCell xử lý row-end termination.

---

## 4.19. Decap Cell

[[DecapCell]] cung cấp local decoupling capacitance.

Mục tiêu:

- cải thiện local power stability;
- giảm voltage droop cục bộ;
- hỗ trợ giảm dynamic [[IRDrop]];
- hỗ trợ power integrity trong standard-cell fabric.

Nhưng:

> DecapCell không thay thế PDN architecture.

Nếu [[PDN]] topology yếu, chỉ chèn thêm DecapCell không giải quyết triệt để power integrity.

---

## 4.20. Filler Cell

[[FillerCell]] lấp khoảng trống giữa các standard cells trong cùng [[Row]].

Mục tiêu:

- fill gap;
- duy trì row physical continuity;
- hỗ trợ well/implant continuity;
- hoàn thiện physical finishing.

[[FillerCell]] thường quan trọng hơn ở giai đoạn sau Placement/physical finishing, nhưng người học nên biết sớm vì nó thuộc hệ physical-only cells của row-based implementation.

Cần phân biệt:

- [[EndCapCell]]: row-end termination.
- [[TapCell]]: well/substrate bias.
- [[DecapCell]]: local decoupling.
- [[FillerCell]]: row gap filling / continuity.
- [[TieCell]]: logic constant tie-off.

---

## 4.21. Bước 5 — Build Power Grid / PDN

[[PDN]] là mạng phân phối nguồn VDD/VSS trong layout.

Trong Floorplanning, power planning thường bao gồm:

- core ring;
- block/macro rings;
- power straps / stripes;
- standard-cell rails;
- macro PG connections;
- IO/core power connection strategy;
- voltage-domain considerations nếu có;
- power/ground net connection context.

PDN phải cấp nguồn liên tục cho:

- standard cells;
- macros;
- IO cells nếu full-chip;
- tie cells;
- clock network;
- high-switching logic.

### Cấu trúc khái niệm của PDN

```text
Package / pads / bumps
  ↓
PG pads / IO ring
  ↓
Core ring
  ↓
Power straps / stripes
  ↓
Standard-cell rails
  ↓
Cell VDD/VSS pins
```

Với macros:

```text
Power straps / rings
  ↓
Macro PG pins
```

### Trade-off của PDN

PDN càng robust:

- IR drop risk giảm;
- EM risk giảm;
- power stability tốt hơn.

Nhưng PDN quá dày:

- chiếm routing resources;
- giảm tài nguyên cho signal routing;
- tăng congestion;
- có thể làm routing/timing khó hơn.

Vì vậy PDN là trade-off giữa power integrity và routability.

---

## 4.22. IR Drop trong Floorplanning

[[IRDrop]] là sụt áp trên mạng nguồn do dòng điện chạy qua điện trở hữu hạn.

Công thức cơ bản:

```text
V_drop = I × R
```

Nếu dòng tăng hoặc điện trở hiệu dụng của PDN cao, voltage drop tăng.

### Static IR Drop

Static IR drop liên quan đến dòng trung bình hoặc tương đối ổn định.

Ở mức khái niệm:

```text
Static IR Drop = I_avg × R
```

Power trung bình gồm:

```text
Average Power = Leakage Power + Internal Power + Switching Power
```

Nếu static IR drop cao, effective VDD tại cell thấp hơn, cell có thể chậm hơn, timing margin giảm.

### Dynamic IR Drop

Dynamic IR drop là sụt áp tức thời do switching activity đồng thời.

Nó thường xuất hiện khi nhiều cells switching gần clock edge, gây current spike.

Hậu quả:

- voltage droop trên VDD;
- ground bounce trên VSS;
- cell delay thay đổi theo thời điểm;
- setup/hold risk có thể tăng.

### IR Drop ảnh hưởng timing như thế nào?

Nếu IR drop xảy ra trên data path:

```text
VDD thấp hơn
  ↓
Cell drive yếu hơn
  ↓
[[CellDelay]] tăng
  ↓
Setup slack xấu hơn
```

Nếu IR drop xảy ra trên clock branch:

```text
Clock buffer chậm hơn
  ↓
Clock arrival thay đổi
  ↓
[[ClockSkew]] thay đổi
  ↓
Hold/setup risk có thể thay đổi
```

Vì vậy Floorplanning phải xem PDN/IR Drop là một phần của timing feasibility, không phải chỉ là power problem.

---

## 4.23. Bước 6 — Floorplan Quality Checks

Sau khi tạo floorplan, cần kiểm tra chất lượng trước khi đi tiếp.

Không nên chuyển sang [[Placement]] chỉ vì tool đã tạo được floorplan.

Các nhóm check chính:

### 1. Power Grid Integrity

Kiểm tra:

- standard-cell rails có connect đúng không;
- macro PG pins có connect không;
- IO/core power có đúng không;
- PG shorts/opens;
- power ring/stripe coverage;
- via connectivity;
- gaps gần macros;
- early IR drop / EM risk.

Nếu PDN sai, các bước sau có thể mất giá trị.

### 2. IO Pads / Pins và Boundary Cells

Kiểm tra:

- IO pad placement hợp lệ;
- block pins đúng vị trí;
- pin layer hợp lý;
- boundary constraints được honor;
- endcap/boundary cells đầy đủ;
- full-chip IO ring có continuity nếu liên quan.

### 3. Macro Placement

Kiểm tra:

- macros đã FIXED chưa;
- macro overlap không;
- macro-to-macro spacing đủ không;
- macro-to-boundary spacing đủ không;
- halo/keep-out hợp lý không;
- macro pins quay đúng hướng không;
- flight lines có crisscross nhiều không;
- standard-cell area có bị chia nát không.

### 4. Placement Rows và Core Utilization

Kiểm tra:

- rows được tạo đầy đủ;
- rows không bị bisect sai bởi macros;
- row orientation hợp lệ;
- placement grid hợp lệ;
- utilization trong target;
- placeable area đủ;
- không tạo vùng standard-cell quá hẹp.

### 5. Blockage Planning

Kiểm tra:

- placement blockages đúng loại;
- routing blockages đúng layer;
- blockages không quá nhiều;
- blockages không quá ít;
- không tạo isolated standard-cell regions;
- không vô tình chặn critical routing corridors.

### 6. General Sanity Checks

Kiểm tra:

- routing tracks;
- manufacturing grid;
- core/die snapping;
- row symmetry;
- constraints;
- pre-routes nếu có;
- guide/region/fence nếu flow dùng;
- tool-specific floorplan checks.

---

## 4.24. Trial Routing và Congestion Analysis

Floorplan không nên được đánh giá chỉ bằng mắt.

Cần trial routing hoặc early global routing để xem:

- routing demand;
- routing supply;
- congestion hotspots;
- macro pin access;
- narrow channels;
- choke points;
- blockages có hợp lý không;
- PDN có chiếm quá nhiều routing resources không.

Nếu congestion fail nặng ở stage này, nên sửa floorplan ngay.

Không nên chờ đến detailed routing mới phát hiện floorplan không routable.

Một câu cần nhớ:

> Floorplanning phải nghĩ routing trước, placement sau.

Điều này không có nghĩa timing không quan trọng. Nó có nghĩa là mọi timing-friendly placement cũng vô nghĩa nếu router không có đường đi hợp lệ.

---

## 4.25. Netlist Uniquification

Netlist uniquification là khái niệm dễ bị bỏ qua.

Nếu cùng một module được instantiate nhiều lần, tool có thể cần tạo bản sao vật lý độc lập cho từng instance để optimization cục bộ không ảnh hưởng lẫn nhau.

Ví dụ:

```text
ALU_1 và ALU_2 cùng reference module ALU
```

Nếu không uniquify, việc chèn buffer vào `ALU_1` có thể vô tình ảnh hưởng logic/reference dùng bởi `ALU_2`, tùy cách database/hierarchy được xử lý.

Ý nghĩa trong Floorplanning:

> Trước khi optimization vật lý cục bộ, mỗi physical instance cần có khả năng được xử lý độc lập khi flow yêu cầu.

Không cần đi sâu command ở đây. Chỉ cần hiểu đây là bước chuẩn bị database để local physical optimization an toàn hơn.

---

## 4.26. Bước 7 — Go to Placement

Floorplan chỉ nên handoff sang [[Placement]] khi các điều kiện chính đạt.

Exit criteria gợi ý:

```text
Core boundary hợp lệ
Aspect ratio và utilization hợp lý
Rows / PlacementGrid hợp lệ
Macro positions đã FIXED
Macro halos / keep-outs hợp lý
No severe congestion in trial routing
PDN connected ở mức floorplan
Early IR Drop feasible
IO/pin planning hợp lệ
PlacementBlockage / RoutingBlockage hợp lý
No major floorplan DRC/sanity issue
Timing feasibility không quá xấu
```

Output truyền xuống [[Placement]]:

```text
Core boundary
Macro fixed positions
Rows
PlacementGrid
Placeable standard-cell regions
Placement blockages
Routing blockages
PDN context
Floorplan DEF / database checkpoint
```

Nếu Floorplanning fail, phải sửa ở Floorplanning. Không nên đẩy vấn đề xuống Placement với hy vọng tool tự cứu.

---

## 4.27. Floorplanning ảnh hưởng các bước sau như thế nào?

### Ảnh hưởng tới Placement

[[Placement]] bị giới hạn bởi:

- core area;
- rows;
- placement grid;
- macro fixed positions;
- placement blockages;
- available whitespace;
- utilization target.

Floorplan quá chật → Placement thiếu không gian tối ưu.  
Floorplan quá phân mảnh → Placement khó giảm wire length và congestion.  
Macro đặt xấu → standard-cell area bị cắt vụn.

### Ảnh hưởng tới CTS

[[ClockTreeSynthesis]] bị ảnh hưởng bởi:

- khoảng cách clock source đến sinks;
- vị trí các clocked macros;
- vị trí register clusters;
- core shape;
- blockage và routing resources;
- power integrity của clock buffers.

Macro clocked như SRAM đặt quá xa clock source có thể làm [[ClockSkew]] và [[ClockLatency]] khó kiểm soát.

### Ảnh hưởng tới Routing

[[Routing]] bị ảnh hưởng bởi:

- routing channels giữa macros;
- [[RoutingGrid]];
- macro obstructions;
- placement density;
- routing blockages;
- PDN metal usage;
- pin access;
- IO/block pin distribution.

Routing fail do floorplan thường phải quay lại floorplan, không chỉ sửa router settings.

### Ảnh hưởng tới PowerAnalysis và Signoff

[[PowerAnalysis]] và [[Signoff]] bị ảnh hưởng bởi:

- PDN topology;
- strap width/spacing;
- macro PG connection;
- local power hotspots;
- standard-cell density;
- dynamic IR risk;
- EM risk;
- DRC/LVS context.

Nếu [[IRDrop]] fail nghiêm trọng, fix có thể yêu cầu thay đổi PDN hoặc floorplan, tức là quay lại rất sớm trong flow.

---

## 4.28. Các lỗi floorplan phổ biến của người mới

### Lỗi 1 — Chỉ tối ưu area

Sai:

> Ép utilization càng cao càng tốt để tiết kiệm area.

Đúng:

> Utilization phải chừa routing resources và optimization whitespace. Area nhỏ nhưng không route được là floorplan thất bại.

---

### Lỗi 2 — Chỉ nhìn macro bằng mắt

Sai:

> Macro đặt nhìn gọn là tốt.

Đúng:

> Macro placement phải được kiểm tra bằng connectivity, flight lines, routing channels, pin access, clock, PDN và trial routing.

---

### Lỗi 3 — Nhầm Placement Blockage với Routing Blockage

Sai:

> Đặt placement blockage là đã cấm route.

Đúng:

> [[PlacementBlockage]] điều khiển cell placement. [[RoutingBlockage]] điều khiển routing resource trên layer cụ thể.

---

### Lỗi 4 — Đặt macros quá sát nhau

Sai:

> Macros liên quan nên đặt sát nhau nhất có thể.

Đúng:

> Đặt gần giúp timing/wirelength, nhưng vẫn phải chừa routing corridor, pin access, PG tapping và tránh chimney/notch.

---

### Lỗi 5 — Xem PDN là bước phụ

Sai:

> PDN để sau routing/signoff mới quan tâm.

Đúng:

> [[PDN]] được xây từ Floorplanning. Nếu power architecture sai, [[IRDrop]] và EM có thể làm fail toàn bộ flow.

---

### Lỗi 6 — Dùng DecapCell thay cho PDN

Sai:

> IR drop cao thì chỉ cần thêm [[DecapCell]].

Đúng:

> DecapCell hỗ trợ local decoupling, nhưng không thay thế PDN architecture. Nếu PDN yếu, phải sửa topology/straps/rings/connectivity.

---

### Lỗi 7 — Không phân biệt full-chip và block-level

Sai:

> IO pad, block pin và LEF pin là cùng một chuyện.

Đúng:

> Full-chip IO pad, block-level boundary pin và LEF cell pin là các khái niệm liên quan nhưng khác scope.

---

### Lỗi 8 — Tin floorplan pass chỉ vì tool không báo fatal error

Sai:

> Không có fatal error nghĩa là floorplan ổn.

Đúng:

> Floorplan phải pass quality checks: congestion, rows, macro placement, blockages, PDN, IO/pins, IR feasibility và timing feasibility.

---

## 4.29. Checklist học tập cho chương 4

Sau chương này, người học nên tự trả lời được:

1. [[Floorplanning]] khác gì với [[DesignImport]]?
2. Vì sao Floorplanning là bước có leverage cao nhất trong PD flow?
3. [[CoreArea]] được ước tính từ những yếu tố nào?
4. Vì sao target utilization không nên quá cao?
5. Core utilization khác standard-cell utilization như thế nào?
6. Aspect ratio ảnh hưởng routing và clock như thế nào?
7. Full-chip Floorplanning khác block-level Floorplanning ở đâu?
8. [[IOCell]] khác block pin như thế nào?
9. Vì sao [[MacroPlacement]] khó hơn standard-cell placement?
10. Flight-line analysis dùng để làm gì?
11. Macro orientation ảnh hưởng routing như thế nào?
12. Vì sao standard-cell area cần liên tục?
13. Halo khác placement blockage thông thường ở đâu?
14. [[PlacementBlockage]] khác [[RoutingBlockage]] như thế nào?
15. [[TapCell]], [[EndCapCell]], [[DecapCell]], [[FillerCell]] khác nhau ở đâu?
16. [[PDN]] gồm các tầng khái niệm nào?
17. [[IRDrop]] ảnh hưởng timing như thế nào?
18. Floorplan quality check gồm những nhóm nào?
19. Khi trial routing báo congestion nặng, nên sửa ở đâu?
20. Khi nào floorplan đủ điều kiện handoff sang [[Placement]]?

---

## 4.30. Quan hệ với các chương trước và sau

Chương 1 đã giới thiệu các input:

```text
[[GateLevelNetlist]]
[[SDC]]
[[LIB]]
[[LEF]]
[[DEF]]
[[MMMC]]
[[PhysicalConstraints]]
```

Chương 2 đã giải thích geometry foundation:

```text
[[LEF]] → [[Site]] → [[Row]] → [[PlacementGrid]]
[[LEF]] → [[Pitch]] → [[Track]] → [[RoutingGrid]]
[[CellAbstract]] → [[Pin]] + [[Obstruction]]
```

Chương 3 đã kiểm tra và tích hợp input:

```text
[[DesignImport]]
  ↓
Validated design database
```

Chương 4 dùng validated database đó để tạo physical plan:

```text
Validated design database
  ↓
[[Floorplanning]]
  ↓
Core boundary + macros + rows + blockages + PDN
  ↓
[[Placement]]
```

Chương tiếp theo là:

```text
# Chương 5 — Placement
```

Ở chương 5, người học sẽ học cách tool đặt [[StandardCell]] vào [[PlacementGrid]], tối ưu wirelength/timing/congestion, chạy [[GlobalPlacement]], [[Legalization]], [[DetailedPlacement]], [[DRVFixing]], [[PreCTSOptimization]], [[CongestionAnalysis]] và chuẩn bị handoff sang [[ClockTreeSynthesis]].

---

## 4.31. Tóm tắt chương 4

[[Floorplanning]] là bước biến validated logical/physical database thành physical implementation plan.

7 sub-steps chính:

```text
1. Define Boundary
2. Place IOs / Assign Block Pins
3. Place Macros
4. Define Standard Cell Areas
5. Build Power Grid
6. Check Quality
7. Go to Placement
```

Output chính:

```text
[[CoreArea]]
IO pad / block pin positions
[[MacroPlacement]]
Macro FIXED locations
[[PlacementBlockage]]
[[RoutingBlockage]]
[[Row]]
[[PlacementGrid]]
[[RoutingGrid]]
[[PDN]]
Early [[IRDrop]] feasibility
Floorplan checkpoint / [[DEF]]
```

Floorplanning constrain các bước sau:

```text
[[Floorplanning]]
  ↓
[[Placement]]
  ↓
[[ClockTreeSynthesis]]
  ↓
[[Routing]]
  ↓
[[ParasiticExtraction]]
  ↓
[[Signoff]]
```

Câu cần nhớ:

> Một floorplan tốt không chỉ fit được design vào core. Nó phải tạo ra một physical search space đủ rộng, đủ sạch và đủ routable để Placement, CTS, Routing và Signoff có thể thành công.