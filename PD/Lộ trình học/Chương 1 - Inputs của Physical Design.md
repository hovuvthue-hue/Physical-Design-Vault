## 1.1. Mục tiêu của chương

Sau chương này, người học cần hiểu:

- Physical Design không bắt đầu từ RTL.
- Physical Design bắt đầu khi logic đã được synthesize thành [[GateLevelNetlist]].
- PnR tool không chỉ cần netlist; nó cần nhiều lớp dữ liệu khác nhau: timing, physical, library, constraint, mode/corner và design-state.
- Nếu input sai hoặc thiếu, các bước sau như [[Floorplanning]], [[Placement]], [[ClockTreeSynthesis]], [[Routing]] và [[Signoff]] có thể tạo ra kết quả vô nghĩa dù tool vẫn chạy được.

Một câu cần nhớ:

> Physical Design là quá trình biến logical connectivity thành physical geometry, rồi chứng minh geometry đó vẫn đúng về timing, power, routability và manufacturing rules.

## 1.2. Physical Design nằm ở đâu trong back-end flow?

Trong digital back-end flow, có thể chia đơn giản như sau:

```
RTL
  ↓
Logic Synthesis
  ↓
Gate-Level Netlist
  ↓
Physical Design / Place and Route
  ↓
Parasitic Extraction
  ↓
Signoff
  ↓
GDS / Tape-out
```

[[LogicSynthesis]] nhận RTL, [[SDC]] và [[LIB]], sau đó tạo ra [[GateLevelNetlist]].
[[Physical Design]] nhận [[GateLevelNetlist]] cùng nhiều file phụ trợ khác để tạo layout vật lý.

Ở mức người mới, cần phân biệt rõ:

[[LogicSynthesis]] trả lời câu hỏi: “Logic này có thể map thành standard cells và có khả năng meet timing với ước lượng sơ bộ không?”
[[Physical Design]] trả lời câu hỏi: “Khi các cells và wires có vị trí vật lý thật, design còn meet timing, power, DRC/LVS và signoff không?”
[[Signoff]] trả lời câu hỏi: “Design đã đủ sạch để release GDS chưa?”

Không nên hiểu Physical Design là “vẽ layout”. Với digital implementation hiện đại, phần lớn công việc là dùng PnR tool để tối ưu và kiểm chứng physical database theo nhiều constraint.

## 1.3. Input tối thiểu của Physical Design

Một Physical Design flow thực tế thường cần các nhóm input sau:

Logical input:
- Gate-level netlist

Timing and constraint input:
- [[SDC]]
- [[MMMC]] setup
- Liberty timing/power libraries ([[LIB]])

Physical and technology input:
- [[LEF]]
- [[DEF]]
- Technology information
- Extraction technology files

Design intent / implementation constraints:
- Physical constraints
- Power intent, nếu flow có UPF/CPF

Trong vault này, các concept nên được đọc theo vai trò, không đọc rời rạc:

- [[GateLevelNetlist]]: mô tả logic đã synthesize.
- [[SDC]]: mô tả timing intent.
- [[LIB]]: mô tả timing/power behavior của cells.
- [[LEF]]: mô tả abstract physical view và technology geometry.
- [[DEF]]: lưu design physical state qua các checkpoint.
- [[MMMC]]: tổ chức modes, corners và analysis views.
- [[PhysicalConstraints]]: ràng buộc hình học và implementation.
- [[DesignImport]]: bước load và kiểm tra các input trên trước khi đi vào floorplan.

---

## 1.4. [[GateLevelNetlist]] — logical connectivity sau synthesis

[[GateLevelNetlist]] là output chính của [[LogicSynthesis]] và là input logic chính của Physical Design.

Nó thường chứa:

- danh sách instance của [[StandardCell]];
- danh sách instance của macro hoặc [[HardIP]], nếu design có;
- nets nối giữa các pins;
- hierarchy, nếu netlist chưa flatten hoàn toàn;
- clock/reset/data connectivity;
- đôi khi có inserted cells từ synthesis như clock gating cells, scan cells hoặc tie-related logic tùy flow.

Điểm quan trọng: [[GateLevelNetlist]] không chứa đủ thông tin vật lý.

Netlist cho biết: Cell A nối với Cell B

nhưng chưa cho biết: 
Cell A nằm ở đâu?
Cell B nằm ở đâu?
Wire đi qua metal layer nào?
Wire dài bao nhiêu?
Wire có congestion không?
Wire có vi phạm DRC không?
Net delay thật là bao nhiêu?

Đây là lý do Physical Design tồn tại. Nó biến connectivity trong netlist thành placement coordinates và routing geometry.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “Có netlist rồi thì chỉ cần đặt cell và route dây.”

Cách hiểu đúng:

> Netlist chỉ là connectivity. Physical Design phải quyết định vị trí cell, macro, pin, power grid, clock tree, routing topology, parasitic RC và kiểm chứng toàn bộ chúng qua nhiều bước.

---

## 1.5. [[SDC]] — timing intent của design

[[SDC]] là một trong các input quan trọng nhất vì nó cho tool biết design cần chạy như thế nào.

SDC thường mô tả:

- clock chính bằng `create_clock`;
- generated clock bằng `create_generated_clock`;
- input delay;
- output delay;
- clock uncertainty;
- false path;
- multicycle path;
- max transition;
- max capacitance;
- max fanout;
- clock groups;
- timing exceptions.

Nếu [[GateLevelNetlist]] cho biết “mạch được nối như thế nào”, thì [[SDC]] cho biết “mạch cần được phân tích theo điều kiện timing nào”.

Ví dụ, cùng một netlist nhưng SDC khác nhau có thể dẫn đến kết quả optimization khác nhau:

- clock period chặt hơn → tool phải tối ưu timing mạnh hơn;
- false path đúng → tool không lãng phí effort vào path không cần timing;
- false path sai → có thể che mất timing violation thật;
- input/output delay sai → interface timing bị phân tích sai;
- clock uncertainty quá thấp → timing report quá lạc quan;
- clock uncertainty quá cao → tool over-optimize và tăng area/power không cần thiết.

### Vai trò của SDC trong Physical Design

[[SDC]] không chỉ dùng ở cuối flow. Nó được dùng từ rất sớm:

Design Import
  ↓ uses SDC
Floorplanning
  ↓ uses timing intent indirectly
Placement
  ↓ uses timing-driven optimization
CTS
  ↓ uses clock definitions and clock constraints
Routing
  ↓ preserves timing/routing constraints
Signoff
  ↓ uses final MMMC timing analysis

Vì vậy, nếu [[SDC]] sai, lỗi sẽ lan qua toàn bộ flow.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “SDC là file của STA, để cuối flow mới cần.”

Cách hiểu đúng:

> SDC là timing contract của toàn bộ implementation flow. Placement, CTS, routing optimization và signoff đều phụ thuộc vào nó.

---

## 1.6. [[LIB]] — timing và power model của standard cells

[[LIB]] hoặc Liberty file mô tả behavior của cells ở mức timing và power.

Nó thường chứa:

- cell pin definitions;
- timing arcs;
- setup/hold constraints cho sequential cells;
- cell delay lookup tables;
- output transition lookup tables;
- input capacitance;
- max transition / max capacitance;
- leakage power;
- internal power;
- power tables;
- operating condition.

Trong [[STA]], [[LIB]] là nguồn dữ liệu gốc để tính [[CellDelay]].  
Một stage delay cơ bản có thể hiểu là:
[[StageDelay]] = [[CellDelay]] + [[NetDelay]]

Trong đó:

- [[CellDelay]] chủ yếu đến từ [[LIB]];
- [[NetDelay]] đến từ wire RC estimate hoặc [[SPEF]] sau extraction.

Trước routing, net delay thường chỉ là ước lượng. Sau [[ParasiticExtraction]], [[SPEF]] cung cấp RC thật hơn cho STA.

### Tại sao LIB quan trọng trong Physical Design?

Physical Design tool cần [[LIB]] để biết:

- cell nào nhanh/chậm;
- cell nào drive mạnh/yếu;
- cell nào có leakage cao/thấp;
- cell nào có setup/hold requirement;
- cell nào phù hợp để resize, buffer, clone hoặc swap threshold voltage;
- transition/cap/fanout có vi phạm design rule hay không.

Nếu thiếu hoặc dùng sai [[LIB]], timing/power optimization sẽ không đáng tin.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “LEF cho cell, vậy đủ rồi.”

Cách hiểu đúng:

> [[LEF]] cho physical abstract. [[LIB]] cho timing/power behavior. PnR cần cả hai.

---

## 1.7. [[LEF]] — abstract physical view của technology và cells

[[LEF]] là input vật lý nền tảng của Physical Design.

Có thể chia đơn giản thành:

- Technology LEF;
- Macro LEF hoặc cell LEF.

Technology LEF mô tả các thông tin như:

- routing layers;
- metal direction;
- width;
- spacing;
- [[Pitch]];
- via definitions;
- manufacturing grid;
- site definitions.

Macro/cell LEF mô tả abstract view của cells/macros:

- cell size;
- pin shapes;
- pin locations;
- obstructions;
- symmetry;
- site compatibility.

Trong vault, các card liên quan trực tiếp gồm:

- [[LEF]]
- [[Pitch]]
- [[Track]]
- [[RoutingGrid]]
- [[Site]]
- [[Row]]
- [[PlacementGrid]]
- [[CellAbstract]]
- [[Pin]]
- [[Obstruction]]
- [[StandardCell]]
- [[HardIP]]

### LEF không phải layout đầy đủ

[[LEF]] không chứa full transistor-level layout. Nó là abstract physical representation để PnR tool dùng.

Ví dụ, với một standard cell, PnR tool không cần biết toàn bộ diffusion/poly/internal device geometry. Nó cần biết:

- cell rộng bao nhiêu;
- cell cao bao nhiêu;
- pins ở đâu;
- các vùng metal internal nào không được route xuyên qua;
- cell đặt được trên site/row nào.

Full layout thật thường nằm trong [[GDS]], còn [[LEF]] là bản rút gọn phục vụ implementation.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “LEF là layout.”

Cách hiểu đúng:

> LEF là physical abstract cho PnR. GDS mới là layout geometry đầy đủ dùng cho manufacturing/signoff.

---

## 1.8. [[DEF]] — carrier của design physical state

[[DEF]] là file mô tả design-specific physical state.

Nếu [[LEF]] mô tả “library/technology có gì”, thì [[DEF]] mô tả “design hiện tại đang như thế nào”.

DEF có thể chứa:

- die area;
- core area;
- rows;
- tracks;
- components;
- placement coordinates;
- blockages;
- pins;
- nets;
- special nets;
- routing information;
- floorplan/placement/routing checkpoint tùy stage.

Trong Physical Design flow, [[DEF]] thường được cập nhật qua nhiều checkpoint:

Post-floorplan DEF
  ↓
Post-placement DEF
  ↓
Post-CTS DEF
  ↓
Post-route DEF

Điểm quan trọng: [[DEF]] không phải chỉ là output cuối. Nó là design-state carrier giữa các bước.

### Ví dụ trực giác

Sau [[Floorplanning]], DEF có thể chứa:

- core boundary;
- rows;
- macro fixed locations;
- placement blockages;
- routing blockages;
- initial power structures.

Sau [[Placement]], DEF có thêm:

- standard cell coordinates;
- legalized placement;
- possibly inserted buffers/cells.

Sau [[Routing]], DEF có thêm:

- detailed routed nets;
- via information;
- routing shapes.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “DEF là file route.”

Cách hiểu đúng:

> DEF là file trạng thái vật lý của design. Nó có thể chứa floorplan, placement và routing information tùy checkpoint.

---

## 1.9. [[MMMC]] — multi-mode multi-corner framework

[[MMMC]] là framework để phân tích design trong nhiều điều kiện hoạt động.

MMMC thường tổ chức các lớp như:

Library Set
  ↓
RC Corner
  ↓
Delay Corner
  ↓
Constraint Mode
  ↓
Analysis View

Ý nghĩa đơn giản:

- Multi-mode: design có thể chạy trong nhiều mode khác nhau, ví dụ functional mode, scan mode, low-power mode.
- Multi-corner: design phải pass ở nhiều PVT/RC corner khác nhau.
- Analysis view: tổ hợp giữa mode và corner dùng cho setup/hold/power analysis.

Ví dụ trực giác:

- Setup thường khó ở slow corner, high RC, high temperature.
- Hold thường khó ở fast corner, low RC, low temperature.

Không nên xem MMMC là phần phụ. MMMC quyết định design được kiểm chứng trong những điều kiện nào.

### Vai trò trong Physical Design

[[MMMC]] ảnh hưởng đến:

- timing analysis;
- placement optimization;
- CTS target;
- hold fixing;
- post-route timing;
- signoff closure.

Một design có thể clean ở một corner nhưng fail ở corner khác. Vì vậy, Physical Design không chỉ tối ưu một timing number duy nhất.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “Chỉ cần WNS/TNS tốt ở một report là design ổn.”

Cách hiểu đúng:

> Cần biết report đó thuộc analysis view nào. Signoff yêu cầu clean trên các views cần thiết, không phải một view đơn lẻ.

---

## 1.10. [[PhysicalConstraints]] — ràng buộc vật lý của implementation

[[PhysicalConstraints]] là nhóm ràng buộc cho biết design được phép tồn tại trong không gian vật lý nào.

Nó có thể bao gồm:

- die size;
- core area;
- aspect ratio;
- block boundary;
- pin locations;
- macro placement constraints;
- placement blockages;
- routing blockages;
- keepout/halo;
- power-grid constraints;
- voltage-area constraints nếu có multi-voltage domain;
- package/I/O constraints nếu là chip-level design.

Nếu [[SDC]] là timing contract, thì [[PhysicalConstraints]] là geometry/implementation contract.

### Block-level và full-chip khác nhau

Ở block-level Physical Design:

- boundary thường được top-level floorplan định nghĩa;
- I/O pins là block pins;
- không có pad ring đầy đủ;
- macro placement và pin placement chịu ràng buộc từ integration level cao hơn.

Ở full-chip Physical Design:

- phải xử lý pad ring;
- I/O cells;
- package constraints;
- ESD;
- seal ring;
- bump hoặc bond pad;
- die/core relationship;
- power delivery từ chip boundary vào core.

Vì vậy, khi học [[Floorplanning]], phải biết mình đang nói về block-level hay full-chip level. Nếu không phân biệt, rất dễ trộn lẫn [[Pin]], block I/O pin và pad cell.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “Floorplan chỉ là chọn core size.”

Cách hiểu đúng:

> Floorplanning là xác định toàn bộ physical search space: boundary, rows, macros, pins, blockages, PDN và feasibility cho các bước sau.

---

## 1.11. [[DesignImport]] — bước load và kiểm tra input

[[DesignImport]] là bước đầu của PnR flow.

Nhiệm vụ chính:

1. Load [[GateLevelNetlist]].
2. Load [[LEF]].
3. Load [[LIB]].
4. Load [[SDC]].
5. Load [[MMMC]].
6. Load [[PhysicalConstraints]] hoặc initial DEF nếu có.
7. Initialize design database.
8. Kiểm tra chất lượng input.
9. Chạy timing sanity check ban đầu.

Ở bước này, tool chưa thật sự tối ưu layout. Mục tiêu là xác nhận design có đủ điều kiện đi tiếp.

### Các lỗi thường gặp ở Design Import

Một số lỗi phổ biến:

- missing library cell;
- black box unresolved;
- mismatch giữa netlist và LEF/LIB;
- missing clock definition;
- unconstrained endpoints;
- invalid SDC;
- missing macro LEF;
- missing power/ground setup;
- wrong MMMC view;
- inconsistent unit;
- bad hierarchy reference.

Nếu [[DesignImport]] fail, không nên cố đi tiếp sang [[Floorplanning]].  
Lý do: floorplan trên một database sai sẽ làm các bước sau mất ý nghĩa.

### Exit criteria gợi ý

Trước khi đi tiếp, người học nên hiểu các câu hỏi kiểm tra:

- Design có black box không?
- Tất cả standard cells có LEF/LIB view tương ứng không?
- Tất cả macros có abstract view không?
- Clock đã được define chưa?
- Timing paths chính có bị unconstrained không?
- MMMC views có được activate đúng không?
- Power/ground context có hợp lệ không?
- Có lỗi fatal nào trong import log không?

---

## 1.12. Tổng kết chương 1

Physical Design không thể bắt đầu chỉ với netlist. Nó cần một tập input đồng bộ:

[[GateLevelNetlist]]  → logic structure
[[SDC]]               → timing intent
[[LIB]]               → cell timing/power model
[[LEF]]               → physical abstract and technology geometry
[[DEF]]               → design physical state
[[MMMC]]              → analysis modes and corners
[[PhysicalConstraints]] → geometry and implementation constraints

Quan hệ quan trọng:

[[GateLevelNetlist]] + [[SDC]] + [[LIB]] + [[LEF]] + [[MMMC]] + [[PhysicalConstraints]]
  ↓
[[DesignImport]]
  ↓
Validated design database
  ↓
[[Floorplanning]]

Nếu chương 1 là “tool cần biết những gì”, thì chương 2 là “tool hiểu không gian vật lý như thế nào”.