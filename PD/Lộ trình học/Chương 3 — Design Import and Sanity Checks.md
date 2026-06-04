## 3.1. Mục tiêu của chương

Sau chương này, người học cần hiểu:

- [[DesignImport]] là bước đầu tiên của PnR flow.
- Mục tiêu của Design Import không phải là tối ưu PPA.
- Mục tiêu của Design Import là đọc, tích hợp và kiểm tra toàn bộ input trước khi sang [[Floorplanning]].
- Nếu Design Import fail, không nên cố chạy tiếp.
- Zero-RC timing check và timing coverage là hai gate quan trọng trước floorplan.
- [[GateLevelNetlist]], [[LEF]], [[MMMC]], [[SDC]], [[PhysicalConstraints]], [[CellAbstract]] và power/ground setup phải đồng bộ với nhau.

Một câu cần nhớ:

> Design Import không tạo layout tốt hơn; nó quyết định design database có đủ sạch để bắt đầu Physical Design hay không.

---

## 3.2. Design Import nằm ở đâu trong PnR flow?

Trong lộ trình này, PnR flow bắt đầu như sau:


[[GateLevelNetlist]] + [[LEF]] + [[LIB]] + [[SDC]] + [[MMMC]] + [[PhysicalConstraints]]
  ↓
[[DesignImport]]
  ↓
Validated design database
  ↓
[[Floorplanning]]


[[DesignImport]] là bước chuyển từ “một tập file rời rạc” thành “một design database thống nhất” mà PnR tool có thể xử lý.

Trước Design Import, ta có nhiều loại file:

- netlist file;
- LEF files;
- LIB files;
- MMMC setup;
- SDC constraints;
- IO placement hoặc block boundary constraints;
- power/ground net declarations;
- technology/extraction-related files.

Sau Design Import, tool phải có một database thống nhất chứa:

- hierarchy;
- instances;
- nets;
- cell abstracts;
- timing views;
- design objects;
- physical boundary hoặc initial physical context;
- power/ground net context;
- early timing graph.

Nếu database này sai, mọi bước sau đều bị ảnh hưởng.

---

## 3.3. Vì sao không nên bỏ qua Design Import?

Người mới thường xem Design Import là thao tác “đọc file”. Cách hiểu đó quá nông.

Design Import thực chất là bước kiểm tra xem các nhóm dữ liệu khác nhau có khớp với nhau không.

Ví dụ:

- [[GateLevelNetlist]] nói có instance `U123`.
- [[LEF]] phải có physical abstract cho cell master của `U123`.
- [[LIB]] phải có timing/power model cho cell master đó.
- [[SDC]] phải reference đúng port/pin/net/cell name sau khi netlist được đọc.
- [[MMMC]] phải chỉ đúng LIB, RC corner, SDC mode và analysis view.
- [[PhysicalConstraints]] phải reference đúng IO pad instance hoặc block pin.
- Power/ground net names phải match design convention.
- Nếu có macro hoặc [[HardIP]], tool cần abstract view để biết size, pins và obstructions.

Nếu bất kỳ nhóm nào không khớp, tool có thể báo lỗi trực tiếp hoặc nguy hiểm hơn: một số path/object bị bỏ qua, làm timing report có “blind spot”.

---

## 3.4. Inputs chính của Design Import

Design Import cần nhiều loại input. Không nên học chúng như danh sách file, mà nên học theo vai trò.

### 3.4.1. [[GateLevelNetlist]] — logical structure

[[GateLevelNetlist]] cung cấp:

- hierarchy;
- module instances;
- [[StandardCell]] instances;
- macro hoặc [[HardIP]] instances;
- nets;
- connectivity;
- clock/reset/data connection;
- scan-related logic nếu netlist đã scan inserted;
- tie-related hoặc special cells nếu synthesis/DFT đã tạo.

Vai trò của netlist trong Design Import:

```text
[[GateLevelNetlist]]
  ↓
design hierarchy + instance list + connectivity database
```

Netlist là nguồn cho tool biết “design gồm những gì và chúng nối với nhau ra sao”.

Nhưng netlist chưa cho biết:

- cell nằm ở đâu;
- macro đặt chỗ nào;
- wire dài bao nhiêu;
- clock tree thật ra sao;
- parasitic RC thật là gì;
- routing có DRC-clean không.

Đây là lý do Design Import mới chỉ là điểm bắt đầu, không phải kết quả Physical Design.

---

### 3.4.2. [[LEF]] và [[CellAbstract]] — physical abstract

[[LEF]] cung cấp physical abstraction cho PnR tool.

Trong Design Import, LEF được dùng để tạo physical database của:

- standard cells;
- macro cells;
- pad cells nếu là chip-level;
- routing layers;
- sites;
- tracks;
- pins;
- obstructions;
- technology rules ở mức PnR cần dùng.

Quan hệ chính:

```text
[[LEF]]
  ↓
[[CellAbstract]]
  ↓
cell size + pin locations + obstruction regions
```

Nếu netlist có instance của một cell nhưng LEF không có abstract tương ứng, tool không biết cell đó:

- rộng bao nhiêu;
- cao bao nhiêu;
- pin nằm ở đâu;
- có obstruction nào không;
- được đặt trên site nào.

Khi đó không thể tiến hành [[Floorplanning]], [[Placement]] hoặc [[Routing]] một cách hợp lệ.

---

### 3.4.3. [[LIB]] — timing/power model

[[LIB]] cung cấp timing và power model cho cells.

Trong Design Import, LIB thường được load thông qua [[MMMC]] setup, không nhất thiết đọc rời như một file đơn lẻ trong mọi flow.

LIB cho tool biết:

- [[CellDelay]];
- setup/hold requirement;
- input capacitance;
- output transition;
- max transition;
- max capacitance;
- leakage power;
- internal power;
- timing arcs.

Nếu LEF cho tool biết cell “trông như thế nào về mặt physical abstract”, thì LIB cho tool biết cell “hành xử như thế nào về timing/power”.

Không có LIB đúng, [[STA]] không đáng tin.

---

### 3.4.4. [[SDC]] — timing intent

[[SDC]] cho biết design cần được timing như thế nào.

Nó định nghĩa:

- clock;
- generated clock;
- input delay;
- output delay;
- clock uncertainty;
- timing exceptions;
- false path;
- multicycle path;
- clock groups;
- max transition;
- max capacitance;
- max fanout.

Trong Design Import, SDC thường được đưa vào thông qua [[MMMC]] constraint modes.

Một điểm rất quan trọng:

> SDC phải resolve được design objects sau khi netlist được load.

Nếu SDC reference sai object name, tool có thể báo:

- object not found;
- dropped constraint;
- unconstrained endpoint;
- unconstrained clock;
- missing input/output delay.

Nếu constraints bị drop hoặc paths bị unconstrained, timing report có thể đẹp giả tạo.

---

### 3.4.5. [[MMMC]] — timing analysis framework

[[MMMC]] tổ chức timing analysis theo nhiều mode và corner.

MMMC thường liên kết:

```text
Library Set
  ↓
RC Corner
  ↓
Delay Corner
  ↓
Constraint Mode
  ↓
Analysis View
```

Trong Design Import, MMMC giúp tool biết:

- dùng LIB nào cho từng PVT corner;
- dùng RC corner nào;
- dùng SDC nào cho từng operating mode;
- analysis view nào active cho setup;
- analysis view nào active cho hold;
- timing engine cần phân tích design trong những scenario nào.

Nếu [[MMMC]] sai, tool có thể phân tích sai corner, sai mode hoặc thiếu view cần thiết.

### Ví dụ trực giác

Một design có thể pass ở functional slow setup view nhưng fail ở scan hold view.

Do đó, câu hỏi đúng không phải là:

> WNS có dương không?

Câu hỏi đúng hơn là:

> WNS/TNS thuộc analysis view nào, mode nào, corner nào, và các view cần thiết đã active chưa?

---

### 3.4.6. [[PhysicalConstraints]] — boundary, IO/pin và physical intent

[[PhysicalConstraints]] cung cấp ràng buộc vật lý ban đầu trước floorplan.

Tùy cấp độ design, physical constraints có thể khác nhau.

Ở chip-level:

- die dimensions;
- IO pad placement;
- corner cells;
- pad ring-related constraints;
- package-driven constraints.

Ở block-level:

- PR boundary;
- block pin locations;
- pin side constraints;
- interface placement constraints;
- block shape;
- top-level integration constraints.

Trong Design Import, physical constraints giúp tool biết:

- vùng physical hợp lệ của design;
- IO pad hoặc block pin nằm ở đâu;
- instance names trong `.io` file có match netlist không;
- block boundary có hợp lệ không;
- floorplan có initial context nào không.

Nếu chip-level `.io` file chứa pad instance name không tồn tại trong [[GateLevelNetlist]], import sẽ fail hoặc physical database sai.

---

### 3.4.7. Power/Ground setup — VDD/VSS context

Một điểm đã được thêm vào repo: Design Import không chỉ load boundary và IO, mà còn cần khai báo power/ground net names.

Ví dụ:

```text
Power net:  VDD / VDD_CORE / VDDIO
Ground net: VSS / VSS_CORE / VSSIO
```

Tên cụ thể phụ thuộc design convention.

Power/ground setup giúp tool biết:

- net nào là power;
- net nào là ground;
- standard-cell PG pins connect vào net nào;
- macro PG pins connect vào net nào;
- IO power domain connect như thế nào;
- global net connections có đúng không.

Nếu power/ground context sai, các bước sau như [[PDN]], [[IRDrop]], power analysis và physical verification có thể bị sai từ gốc.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “Power grid là chuyện của Floorplanning, chưa cần ở Design Import.”

Cách hiểu đúng:

> Floorplanning xây [[PDN]], nhưng Design Import phải có power/ground naming và global net context đủ đúng để tool hiểu các PG nets ngay từ đầu.

---

### 3.4.8. [[ITF]] / extraction technology context

Trong vault hiện tại, [[ITF]] xuất hiện trong requirement của [[DesignImport]] và [[MMMC]], thường liên quan đến RC corner hoặc qrcTechFile context.

Ở mức người mới, chỉ cần hiểu:

- [[ITF]] hoặc file extraction technology tương đương mô tả stack/interconnect parameters cần cho RC modeling.
- Nó liên quan đến [[InterconnectRC]], [[NetDelay]], [[ParasiticExtraction]] và [[SPEF]].
- Trong Design Import, nó thường không phải thứ người học tự chỉnh, nhưng cần biết nó là một phần của timing/extraction context.

Không nên nhầm:

- [[LEF]]: physical abstract và routing/placement rules cho PnR.
- [[LIB]]: cell timing/power model.
- [[ITF]] hoặc extraction tech: interconnect RC/extraction modeling context.
- [[SPEF]]: kết quả parasitic extraction sau routing.

---

## 3.5. 7 sub-steps của Design Import

Trong lộ trình này, [[DesignImport]] gồm 7 sub-steps:

```text
1. Netlist Import
2. Cell Abstracts Import
3. Timing Constraints Import
4. Physical Constraints Import
5. Initialize Design
6. Quality of Inputs Checks
7. Quick Timing Checks
```

Không nên học 7 bước này như command sequence. Hãy học theo ý nghĩa dữ liệu.

---

## 3.6. Step 1 — Netlist Import

Mục tiêu:

> Đọc [[GateLevelNetlist]] và xây dựng logical design database.

Tool cần nhận diện:

- top module;
- hierarchy;
- standard-cell instances;
- macro instances;
- nets;
- ports;
- pins;
- unresolved modules;
- duplicate definitions;
- empty modules.

Sau bước này, tool biết design gồm những instance nào và nối với nhau ra sao.

### Các lỗi cần chú ý

- Top module sai.
- Missing module definition.
- Empty module.
- Unresolved reference.
- Duplicate instance.
- Netlist không match library cell names.
- Macro instance có trong netlist nhưng thiếu abstract/library view.

### Black Box

[[Black Box]] là trường hợp một module tồn tại như placeholder nhưng không có implementation đầy đủ.

Trong PnR, Black Box nguy hiểm vì:

- tool không biết physical implementation bên trong;
- timing qua block có thể không được phân tích đúng;
- connectivity có thể không đầy đủ;
- downstream floorplan/placement/routing có thể dựa trên database sai.

Một design có Black Box không nên đi tiếp sang [[Floorplanning]], trừ khi đó là black-boxing có chủ ý và flow đã có model thay thế hợp lệ. Trường hợp này phải được review rõ ràng, không được mặc định là an toàn.

---

## 3.7. Step 2 — Cell Abstracts Import

Mục tiêu:

> Đọc [[LEF]] và tạo physical database cho cells/macros/pads.

Tool cần biết với mỗi cell/macro:

- size;
- site;
- symmetry;
- pin names;
- pin directions;
- pin shapes;
- pin locations;
- obstruction regions;
- routing layers liên quan;
- placement compatibility.

Quan hệ chính:

```text
Netlist instance
  ↓ references
Cell master
  ↓ must have
[[CellAbstract]]
```

Nếu một instance trong netlist không có [[CellAbstract]], tool không thể place hoặc route instance đó một cách đúng.

### Các lỗi cần chú ý

- Missing LEF for standard cell.
- Missing LEF for macro.
- LEF cell name không match netlist master name.
- Pin name trong LEF không match pin name trong logical model.
- Macro obstruction thiếu hoặc sai.
- SITE mismatch.
- Units mismatch.
- Layer naming mismatch.

### Tại sao bước này quan trọng?

Vì từ thời điểm này, tool bắt đầu gắn logical objects vào physical information.

Trước đó, netlist chỉ nói:

```text
U1 NAND2_X1 nối với U2 DFF_X1
```

Sau khi đọc LEF, tool biết thêm:

```text
NAND2_X1 rộng bao nhiêu?
DFF_X1 cao bao nhiêu?
Pins nằm trên layer nào?
Router có thể access pin ở đâu?
Có vùng OBS nào không?
```

---

## 3.8. Step 3 — Timing Constraints Import

Mục tiêu:

> Đọc [[MMMC]] / [[SDC]] và tạo timing analysis context.

Tool cần khởi tạo:

- library sets;
- RC corners;
- delay corners;
- constraint modes;
- analysis views;
- active setup views;
- active hold views;
- clock definitions;
- timing exceptions;
- input/output delay constraints.

Ở bước này, không chỉ “đọc SDC”. Tool phải kiểm tra SDC có reference đúng design objects sau khi netlist được load không.

### Object resolution

SDC làm việc trên design objects như:

- design;
- cell;
- reference;
- port;
- pin;
- net;
- clock.

Nếu SDC gọi nhầm object, constraint có thể không được áp dụng.

Ví dụ lỗi thường gặp:

```text
get_ports clk
```

nhưng design thực tế dùng port name khác:

```text
clock
```

Khi đó clock có thể không được define đúng. Các endpoints liên quan có thể trở thành unconstrained.

### Timing exceptions phải có chủ ý

Các timing exceptions như false path hoặc multicycle path không phải là cách “xóa lỗi timing”.

Chúng chỉ hợp lệ nếu phản ánh đúng design intent.

Ví dụ:

- false path đúng: hai clock domains asynchronous thật sự không cần timing synchronous.
- false path sai: che mất một path đáng lẽ phải được check.
- multicycle path đúng: protocol thật sự cho phép nhiều chu kỳ.
- multicycle path sai: che giấu một one-cycle path bị chậm.

### Người mới thường hiểu sai

Sai lầm phổ biến:

> “Nếu report không còn violation sau khi set false path thì design ổn.”

Cách hiểu đúng:

> Timing exception không sửa hardware. Nó chỉ thay đổi cách STA kiểm tra path. Nếu exception sai, report đẹp nhưng silicon có thể lỗi.

---

## 3.9. Step 4 — Physical Constraints Import

Mục tiêu:

> Load physical intent trước khi initialize design database.

Nội dung có thể gồm:

- chip-level die dimensions;
- IO pad placement;
- block-level PR boundary;
- block pin locations;
- pin side constraints;
- macro-related initial constraints;
- blockage intent;
- power/ground net naming;
- global net connection context.

### Chip-level context

Ở chip-level, tool có thể cần file `.io` hoặc constraint tương đương để biết:

- IO pad nào nằm cạnh top;
- IO pad nào nằm cạnh bottom;
- IO pad nào nằm cạnh left/right;
- corner cells ở đâu;
- pad instance names có match netlist không.

Nếu `.io` file reference một pad instance không tồn tại trong netlist, đó là lỗi import.

### Block-level context

Ở block-level, tool có thể cần:

- PR boundary;
- block pin location;
- pin layer;
- pin side;
- boundary shape;
- top-level integration constraints.

Điểm quan trọng:

> Block pin location là hard interface với top-level. Không thể tùy tiện thay đổi nếu chưa đồng bộ với top-level integration.

### Power/Ground naming

Cần khai báo rõ:

- power net names;
- ground net names;
- global net connections;
- macro PG pin mapping;
- standard-cell PG pin mapping;
- IO power/ground nếu có.

Nếu VDD/VSS naming sai, tool có thể không connect đúng PG pins. Lỗi này thường gây hậu quả lớn ở [[PDN]], [[IRDrop]] và signoff physical checks.

---

## 3.10. Step 5 — Initialize Design

Mục tiêu:

> Tích hợp tất cả input thành unified design database.

Sau initialize, tool đã có một database sơ khai gồm:

- logical hierarchy;
- instances;
- nets;
- timing views;
- physical abstracts;
- initial boundary context;
- power/ground context;
- IO/pin context nếu có;
- black boxes hoặc unresolved objects nếu vẫn còn lỗi.

Ở bước này, design chưa có floorplan hoàn chỉnh. Standard cells chưa được placed. Clock tree chưa được build. Routing thật chưa tồn tại.

Nhưng database đã đủ để chạy các sanity checks.

### Trạng thái sau initialize

Có thể hình dung database lúc này như sau:

```text
Logical data:
- hierarchy
- instances
- nets
- ports/pins

Timing data:
- SDC constraints
- clocks
- MMMC views
- LIB timing arcs

Physical data:
- LEF abstracts
- initial boundary / IO / pin context
- power/ground net context
```

Nếu database đã sai ở đây, [[Floorplanning]] sẽ nhận input sai.

---

## 3.11. Step 6 — Quality of Inputs Checks

Mục tiêu:

> Kiểm tra input có đầy đủ, hợp lệ và đồng bộ không.

Đây là bước người mới thường bỏ qua, nhưng trong thực tế nó quyết định flow có đáng chạy tiếp hay không.

Có thể chia Quality of Inputs Checks thành 4 nhóm.

---

### 3.11.1. Netlist quality

Cần kiểm tra:

- có missing module không;
- có unresolved reference không;
- có Black Box không;
- có duplicate instance không;
- top module có đúng không;
- hierarchy có đầy đủ không;
- macro instances có được nhận diện đúng không;
- standard-cell masters có tồn tại trong library không.

Câu hỏi cần trả lời:

```text
Netlist có thể trở thành timing graph và implementation database hoàn chỉnh không?
```

Nếu câu trả lời là không, phải quay lại netlist/synthesis/handoff.

---

### 3.11.2. Library / abstract quality

Cần kiểm tra:

- mọi standard cell trong netlist có LEF không;
- mọi standard cell trong netlist có LIB không;
- macro có LEF không;
- macro có timing model hoặc black-box timing model hợp lệ không;
- pin names giữa logical view và physical abstract có match không;
- SITE compatibility đúng không;
- layer naming có nhất quán không;
- units có nhất quán không.

Câu hỏi cần trả lời:

```text
Mỗi logical cell trong netlist có đủ physical view và timing view không?
```

Nếu thiếu LEF, tool không place/route đúng.  
Nếu thiếu LIB, STA không phân tích đúng.

---

### 3.11.3. Constraint quality

Cần kiểm tra:

- clock đã được define chưa;
- generated clocks có đúng không;
- input delay có đủ không;
- output delay có đủ không;
- false paths có chủ ý không;
- multicycle paths có chủ ý không;
- clock groups có đúng không;
- có unconstrained endpoints không;
- có unconstrained clock endpoints không;
- có dropped constraints không;
- SDC object names có resolve đúng không;
- active analysis views có đúng không.

Câu hỏi cần trả lời:

```text
STA có đang kiểm tra đúng những path cần kiểm tra không?
```

Một timing report chỉ đáng tin nếu constraint coverage đủ tốt.

---

### 3.11.4. Physical quality

Cần kiểm tra:

- IO pad instance names có match netlist không;
- PR boundary có hợp lệ không;
- die/core size ban đầu có khả thi không;
- block pins có vị trí hợp lệ không;
- physical units có nhất quán không;
- power/ground net names có đúng không;
- global net connections có được setup đúng không;
- macro abstracts có đủ không;
- hard constraints có mâu thuẫn không.

Câu hỏi cần trả lời:

```text
Design database có đủ physical context để bắt đầu floorplan không?
```

Nếu physical constraints sai, floorplan có thể sai ngay từ đầu.

---

## 3.12. Step 7 — Quick Timing Checks

Mục tiêu:

> Chạy timing sanity check sớm nhất trước khi đầu tư thời gian vào floorplan.

Ở bước này, design chưa có placement và routing thật. Vì vậy, timing check thường dùng mô hình rất lý tưởng hoặc rất thô.

Trong lộ trình này, checkpoint quan trọng là:

```text
Post-Import STA / zero-RC timing sanity check
```

Ý nghĩa:

- clock thường vẫn là ideal clock;
- real clock tree chưa tồn tại;
- net delay chưa có extracted RC;
- wire delay có thể được bỏ qua hoặc chỉ modeled rất sơ bộ tùy flow;
- timing chủ yếu phản ánh synthesis quality + constraint quality + cell delay.

### Zero-RC timing là gì?

Mô hình đơn giản:

```text
[[StageDelay]] = [[CellDelay]] + [[NetDelay]]

Với zero-RC check:
[[NetDelay]] ≈ 0

Nên:
[[StageDelay]] ≈ [[CellDelay]]
```

Setup slack ở mức trực giác:

```text
Zero-RC Setup Slack =
Required Time - Arrival Time

Arrival Time ≈ t_cq + sum(CellDelay)
```

Nếu design fail timing ngay cả khi net delay gần như bằng 0, đó là tín hiệu mạnh cho thấy vấn đề nằm ở:

- synthesis quality;
- clock period quá chặt;
- wrong SDC;
- missing/mis-modeled constraints;
- wrong library/corner;
- bad architecture;
- long logic depth;
- bad mapping;
- missing timing exception hợp lệ.

Không nên kỳ vọng [[Floorplanning]], [[Placement]] hoặc [[Routing]] downstream tự “cứu” một design đã fail nặng ở zero-RC. Downstream physical effects thường làm timing khó hơn, không dễ hơn.

---

## 3.13. Timing coverage check

Timing coverage check trả lời câu hỏi:

```text
Có bao nhiêu timing endpoints thật sự được STA kiểm tra?
```

Một design có WNS đẹp nhưng coverage thấp là nguy hiểm.

Ví dụ:

```text
WNS = +0.20 ns
Coverage = thiếu nhiều input/output paths
```

Kết luận đúng không phải là “timing sạch”.  
Kết luận đúng là:

> Timing report chưa đủ đáng tin vì còn blind spots.

Các loại vấn đề coverage:

- untested setup paths;
- untested hold paths;
- missing clock;
- missing input delay;
- missing output delay;
- unconstrained endpoints;
- disabled paths ngoài ý muốn;
- false paths quá rộng;
- generated clock thiếu hoặc sai.

### Metric cần nhớ

```text
WNS/TNS chỉ có ý nghĩa khi timing coverage đủ tốt.
```

Nếu nhiều path chưa được test, WNS/TNS có thể đang bỏ qua lỗi thật.

---

## 3.14. WNS, TNS và Slack Histogram ở Design Import

Sau quick timing check, các metrics cơ bản gồm:

- [[Slack]];
- WNS;
- TNS;
- number of violating paths;
- timing coverage;
- slack histogram.

### WNS

```text
WNS = Worst Negative Slack
```

WNS cho biết path tệ nhất đang âm bao nhiêu.

Nếu WNS = -0.20 ns, path xấu nhất thiếu 0.20 ns timing margin.

### TNS

```text
TNS = Total Negative Slack
```

TNS cho biết tổng lượng slack âm trên các violating paths.

Một WNS hơi âm nhưng TNS rất lớn có nghĩa là không chỉ một path lỗi, mà rất nhiều paths đang xấu.

### Slack Histogram

[[Slack]] histogram cho biết phân phối timing toàn design:

- nhiều path gần 0 → design nhạy timing;
- long tail âm → có nhóm path rất xấu;
- nhiều path dương lớn → có margin tốt;
- nhiều path untested → coverage problem.

Ở Design Import, slack histogram không phản ánh physical reality đầy đủ, nhưng nó hữu ích để đánh giá synthesis/constraint health trước floorplan.

---

## 3.15. Exit Criteria của Design Import

Trong lộ trình này, [[DesignImport]] chỉ nên pass khi các điều kiện chính sau đạt:

```text
No fatal errors
No unresolved references
No unintended Black Boxes
All required LEF/LIB views available
Timing constraints resolve correctly
Timing coverage sufficiently complete
Zero-RC WNS/TNS acceptable for this flow
Power/Ground net context valid
Physical constraints loaded and consistent
Ready for [[Floorplanning]]
```

Theo cách viết nghiêm ngặt trong vault hiện tại:

```text
WNS ≥ 0
TNS ≥ 0
Timing constraint coverage = 100%
No Black Box
No errors
```

Trong thực tế công nghiệp, tiêu chí cụ thể có thể phụ thuộc project, methodology và stage. Nhưng với lộ trình học cho người mới, dùng tiêu chí nghiêm ngặt như trên là tốt vì nó tạo thói quen không chạy tiếp trên database bẩn.

---

## 3.16. Nếu Design Import fail thì sửa ở đâu?

Không phải lỗi nào cũng sửa trong PnR.

Một bảng định hướng:

| Loại lỗi | Nơi cần kiểm tra trước |
|---|---|
| Missing module / unresolved reference | Netlist handoff, synthesis output |
| Black Box ngoài ý muốn | RTL/synthesis/handoff, macro integration |
| Cell master không có LEF | Library setup, LEF list |
| Cell master không có LIB | MMMC/library setup |
| Pin mismatch giữa LEF và logical model | Library view consistency |
| Clock không được define | SDC |
| Unconstrained endpoints | SDC input/output/clock constraints |
| Dropped constraints | SDC object names, hierarchy names |
| Wrong analysis view | MMMC setup |
| IO pad name mismatch | `.io` file và netlist |
| Block pin constraint sai | Physical constraints / top-level integration |
| Power net không connect đúng | PG net naming / global net connection setup |
| Zero-RC timing fail | Synthesis, SDC, library/corner, architecture |

Nguyên tắc:

> Lỗi thuộc input/handoff thì phải sửa ở input/handoff. Không nên dùng physical optimization để che lỗi dữ liệu đầu vào.

---

## 3.17. Những hiểu lầm cần loại bỏ sau chương 3

### Hiểu lầm 1

> “Design Import chỉ là đọc file.”

Sửa lại:

> Design Import là bước tích hợp và kiểm chứng consistency giữa netlist, library, physical abstract, timing constraints, physical constraints và PG context.

---

### Hiểu lầm 2

> “Tool chạy tiếp được nghĩa là input ổn.”

Sửa lại:

> Tool chạy tiếp không đảm bảo input đúng. Có thể vẫn tồn tại unconstrained paths, dropped constraints, missing views hoặc blind spots trong STA.

---

### Hiểu lầm 3

> “WNS/TNS tốt nghĩa là design sạch.”

Sửa lại:

> WNS/TNS chỉ có ý nghĩa khi timing coverage đủ tốt và analysis views đúng. Nếu nhiều paths untested, WNS/TNS đẹp không đáng tin.

---

### Hiểu lầm 4

> “Black Box chỉ là warning.”

Sửa lại:

> Black Box ngoài ý muốn là rủi ro lớn vì tool không có đủ logical/physical/timing information cho phần design đó.

---

### Hiểu lầm 5

> “Zero-RC timing fail thì cứ sang Placement tối ưu tiếp.”

Sửa lại:

> Nếu zero-RC timing fail nặng, vấn đề thường nằm ở synthesis, SDC, library/corner hoặc architecture. Cần debug trước khi floorplan.

---

### Hiểu lầm 6

> “Power/Ground setup để sau khi tạo PDN mới cần.”

Sửa lại:

> PDN được xây ở [[Floorplanning]], nhưng power/ground net naming và global net context cần đúng từ Design Import để database hiểu PG nets.

---

## 3.18. Checklist học tập cho chương 3

Sau khi học chương này, người học nên tự trả lời được các câu sau:

1. [[DesignImport]] khác gì với [[Floorplanning]]?
2. Vì sao PnR tool không thể chỉ cần [[GateLevelNetlist]]?
3. [[LEF]] và [[LIB]] khác nhau ở đâu?
4. [[SDC]] sai có thể làm timing report đẹp giả tạo như thế nào?
5. [[MMMC]] ảnh hưởng gì đến timing analysis?
6. Vì sao Black Box nguy hiểm?
7. Vì sao missing LEF làm placement/routing không hợp lệ?
8. Vì sao missing LIB làm STA không đáng tin?
9. Vì sao IO pad instance names phải match netlist?
10. Vì sao power/ground net naming phải được khai báo sớm?
11. Zero-RC timing check đang kiểm tra điều gì?
12. Timing coverage khác gì với WNS/TNS?
13. Khi Design Import fail, nên sửa trong PnR hay quay lại input/handoff?
14. Điều kiện nào cho phép design đi tiếp sang [[Floorplanning]]?

---

## 3.19. Quan hệ của chương 3 với các chương trước và sau

Chương 1 đã giải thích các input chính:

```text
[[GateLevelNetlist]]
[[SDC]]
[[LIB]]
[[LEF]]
[[DEF]]
[[MMMC]]
[[PhysicalConstraints]]
```

Chương 2 đã giải thích physical geometry foundation:

```text
[[LEF]] → [[Pitch]] → [[Track]] → [[RoutingGrid]]
[[LEF]] → [[Site]] → [[Row]] → [[PlacementGrid]]
[[CellAbstract]] → [[Pin]] + [[Obstruction]]
```

Chương 3 ghép hai phần này lại:

```text
Logical input + Timing input + Physical input
  ↓
[[DesignImport]]
  ↓
Validated design database
```

Chương tiếp theo là:

```text
[[Floorplanning]]
```

Và [[Floorplanning]] chỉ có ý nghĩa khi nó nhận được một validated design database từ [[DesignImport]].

---

## 3.20. Tóm tắt chương 3

[[DesignImport]] là gate đầu tiên của PnR flow.

Nó gồm 7 sub-steps:

```text
1. Netlist Import
2. Cell Abstracts Import
3. Timing Constraints Import
4. Physical Constraints Import
5. Initialize Design
6. Quality of Inputs Checks
7. Quick Timing Checks
```

Các input chính:

```text
[[GateLevelNetlist]]
[[LEF]]
[[CellAbstract]]
[[LIB]]
[[SDC]]
[[MMMC]]
[[PhysicalConstraints]]
[[ITF]]
Power/Ground net context
```

Các output chính:

```text
Validated design database
Timing sanity result
Input quality report
Go / no-go decision before [[Floorplanning]]
```

Các gate quan trọng:

```text
No Black Box
No fatal errors
No missing LEF/LIB views
No major unresolved constraints
Timing coverage complete enough
Zero-RC timing acceptable
Power/Ground context valid
Physical constraints consistent
```

Câu cần nhớ:

> Design Import là bước rẻ nhất để phát hiện lỗi handoff. Nếu bỏ qua lỗi ở đây, chi phí debug sẽ tăng mạnh ở Floorplanning, Placement, CTS, Routing hoặc Signoff.