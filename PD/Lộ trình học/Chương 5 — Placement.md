

## 5.1. Mục tiêu của chương

Sau chương này, người học cần hiểu:

- [[Placement]] diễn ra sau [[Floorplanning]] và trước [[ClockTreeSynthesis]].
- Placement không chỉ là “đặt standard cells lên row”.
- Placement là một giai đoạn pre-CTS implementation package, gồm legal standard-cell placement, wirelength optimization, timing-driven placement, congestion-driven placement, DRV cleanup, power/area optimization, HFNS, scan chain reordering, spare cells, MBFF và placement qualification.
- Output của Placement là placed database / placed [[DEF]], trong đó [[StandardCell]] đã có tọa độ vật lý hợp lệ.
- Vị trí clock sinks sau Placement là input cứng cho [[ClockTreeSynthesis]].
- Chất lượng Placement quyết định phần lớn rủi ro timing, congestion, routability và power ở các bước sau.

Một câu cần nhớ:

> [[Floorplanning]] tạo legal search space; [[Placement]] tối ưu cells bên trong search space đó; [[ClockTreeSynthesis]] và [[Routing]] thừa hưởng toàn bộ hậu quả của Placement.

---

## 5.2. Placement nằm ở đâu trong PnR flow?

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

[[Placement]] nhận output từ [[Floorplanning]]:

- [[CoreArea]];
- macro FIXED positions;
- [[Row]];
- [[PlacementGrid]];
- [[PlacementBlockage]];
- [[RoutingBlockage]];
- [[PDN]] context;
- standard-cell legal regions;
- floorplan database / [[DEF]].

Sau Placement, tool tạo ra:

- tọa độ `(x, y)` cho từng [[StandardCell]];
- legalized placement;
- updated placed database;
- placement density map;
- pre-route timing estimate;
- congestion estimate;
- inserted buffers hoặc optimization cells nếu flow cho phép;
- possibly updated netlist sau HFNS / optimization;
- placed [[DEF]] hoặc database checkpoint.

Điểm quan trọng:

> Trước Placement, standard cells chưa có vị trí thật. Sau Placement, mỗi cell đã trở thành một object vật lý trong core.

---

## 5.3. Placement giải bài toán gì?

Placement trả lời câu hỏi:

```text
Mỗi standard cell nên được đặt ở đâu trong core để design có khả năng meet timing, routable, không vi phạm legality, và vẫn giữ PPA hợp lý?
```

Đây là bài toán multi-objective.

Các objective chính:

- giảm wirelength;
- cải thiện timing;
- kiểm soát local density;
- giảm congestion;
- giữ placement legality;
- giảm DRV risk;
- hỗ trợ power optimization;
- chuẩn bị clock sinks tốt cho CTS;
- tạo placed database đủ sạch để handoff sang downstream steps.

Không có một objective đơn lẻ nào đủ để đánh giá placement.

Ví dụ:

- HPWL thấp nhưng density quá cao → routing có thể fail.
- Timing tốt nhưng macro channel nghẽn → detailed routing có thể fail.
- Density đều nhưng critical path quá dài → setup fail.
- Buffer quá nhiều để fix timing → area/power tăng.
- Power giảm bằng HVT/downsize quá mạnh → timing có thể xấu.

Vì vậy Placement luôn là trade-off.

---

## 5.4. Inputs của Placement

Placement cần các input chính sau:

```text
[[Floorplanning]]
[[GateLevelNetlist]]
[[SDC]]
[[MMMC]]
[[LEF]]
[[LIB]]
[[DEF]]
[[PlacementGrid]]
[[Row]]
[[Site]]
[[PlacementBlockage]]
[[RoutingBlockage]]
[[PDN]]
```

Vai trò từng nhóm:

- [[Floorplanning]] cung cấp physical search space.
- [[GateLevelNetlist]] cung cấp danh sách instances và nets.
- [[SDC]] cung cấp timing constraints.
- [[MMMC]] cung cấp setup/hold views và corners/modes.
- [[LEF]] cung cấp cell size, pin locations, site/row/routing grid information.
- [[LIB]] cung cấp delay/power model.
- [[DEF]] có thể mang floorplan state từ bước trước.
- [[PlacementGrid]], [[Row]], [[Site]] định nghĩa legal placement positions.
- [[PlacementBlockage]] giới hạn nơi có thể đặt cells.
- [[RoutingBlockage]] ảnh hưởng routing-resource expectation.
- [[PDN]] ảnh hưởng available routing resources và power integrity context.

Nếu design có multi-voltage domains, flow có thể cần thêm UPF/CPF hoặc power intent tương đương. Trong lộ trình này, chỉ cần hiểu đó là input bổ sung khi design có nhiều power domains; không phải mọi single-voltage design đều cần học sâu ngay.

---

## 5.5. Outputs của Placement

Output chính của Placement:

```text
Placed design database
Legalized standard-cell coordinates
Placed DEF / placement checkpoint
Updated timing estimate
Congestion estimate
Placement density map
Inserted buffers / optimization cells nếu có
Updated netlist nếu flow thay đổi connectivity
Reports: timing, congestion, density, utilization, power, DRV
```

Output này truyền xuống:

```text
[[Placement]]
  ↓
[[ClockTreeSynthesis]]
```

Điểm quan trọng nhất đối với CTS:

```text
Clock sink locations = tọa độ vật lý của clock pins trên flip-flops / sequential cells
```

CTS không thể build clock tree hợp lý nếu chưa biết clock sinks nằm ở đâu. Vì vậy Placement là prerequisite trực tiếp của [[ClockTreeSynthesis]].

---

## 5.6. Placement không chỉ là một bước duy nhất

Trong lộ trình này, nên học Placement như một package gồm nhiều lớp:

```text
Pre-placement sanity
  ↓
[[GlobalPlacement]]
  ↓
Early congestion / routability estimate
  ↓
[[Legalization]]
  ↓
[[DetailedPlacement]]
  ↓
[[HFNS]]
  ↓
[[ScanChainReordering]]
  ↓
[[DRVFixing]]
  ↓
[[PreCTSOptimization]]
  ↓
[[PowerAnalysis]] / [[PowerOptimization]]
  ↓
[[MBFF]] / [[SpareCell]] / tie-related cleanup nếu flow có
  ↓
[[CongestionAnalysis]]
  ↓
Placement qualification
  ↓
[[ClockTreeSynthesis]]
```

Không phải tool/flow nào cũng tách các bước này giống hệt nhau. Nhưng về mặt học thuật và tư duy PD, đây là cách chia tốt cho người mới.

---

## 5.7. Pre-placement sanity

Trước khi chạy placement thật, cần xác nhận floorplan đã đủ sạch.

Các câu hỏi cần kiểm tra:

- Core boundary có hợp lệ không?
- [[Row]] đã được tạo đúng chưa?
- [[PlacementGrid]] có hợp lệ không?
- Macro positions đã FIXED chưa?
- [[PlacementBlockage]] có hợp lý không?
- [[RoutingBlockage]] có làm route quá khó không?
- [[PDN]] có chiếm quá nhiều routing resource không?
- Standard-cell utilization có quá cao không?
- Có vùng standard-cell area quá hẹp không?
- Có macro chimney hoặc notch nghiêm trọng không?
- Timing constraints có active đúng không?
- Có unconstrained paths nghiêm trọng không?
- Power/ground context có ổn không?

Nếu floorplan đã sai, chạy Placement chỉ làm lỗi lộ ra muộn hơn.

Một câu cần nhớ:

> Placement không sửa được một floorplan fundamentally unroutable.

---

## 5.8. Placement object: Standard Cell là movable object chính

Trong Placement, object chính được đặt là [[StandardCell]].

Một [[StandardCell]] có:

- fixed height hoặc một số height variants tùy library;
- width là bội số của [[Site]];
- pins với vị trí cụ thể trong [[LEF]];
- timing/power model trong [[LIB]];
- legal orientations;
- drive-strength variants;
- Vt variants nếu library hỗ trợ;
- input capacitance, output drive, leakage, internal power.

Placement tool phải đặt mỗi standard cell:

- trong legal core region;
- trên legal [[Row]];
- aligned với [[Site]];
- không overlap;
- không vi phạm [[PlacementBlockage]];
- không xâm phạm macro fixed area;
- giữ orientation hợp lệ;
- duy trì đủ điều kiện downstream routing.

### Không nên nhầm với Macro

[[HardIP]] / macro thường đã được đặt trong [[Floorplanning]] và FIXED trước Placement.

Placement chủ yếu tối ưu standard cells quanh các fixed macros đó.

---

## 5.9. Placement legality

Một placement hợp lệ phải thỏa:

```text
Cell nằm trong legal placement area
Cell align đúng [[Site]]
Cell nằm trên [[Row]]
Cell không overlap cell khác
Cell không overlap macro / fixed object
Cell không vi phạm [[PlacementBlockage]]
Cell orientation hợp lệ
Cell không phá row/power rail convention
```

Legal placement không đồng nghĩa với placement tốt.

Ví dụ:

- Legal nhưng wirelength dài → timing xấu.
- Legal nhưng density cục bộ cao → congestion.
- Legal nhưng pin access khó → routing fail.
- Legal nhưng clock sinks phân tán xấu → CTS khó.

Vì vậy Placement cần vừa legal vừa tối ưu QoR.

---

## 5.10. [[GlobalPlacement]]

[[GlobalPlacement]] là pha placement thô.

Mục tiêu:

- tìm vị trí tương đối tốt cho cells;
- giảm wirelength proxy;
- cải thiện timing posture;
- phân bố density hợp lý;
- giảm congestion risk;
- tạo nghiệm ban đầu cho legalization/detailed placement.

Ở giai đoạn này, tool có thể cho phép overlap tạm thời giữa cells.

Điều này không có nghĩa output đã hợp lệ. Nó chỉ là trạng thái trung gian để optimizer tìm nghiệm tốt.

### Vì sao Global Placement có thể cho overlap?

Nếu ép legality tuyệt đối ngay từ đầu, không gian tối ưu rất khó tìm. Cho phép overlap tạm thời giúp optimizer tìm vị trí tương đối tốt theo các objective như wirelength, density và timing. Sau đó [[Legalization]] sẽ đưa placement về trạng thái hợp lệ.

### Output của Global Placement

```text
Approximate cell coordinates
Density distribution ban đầu
HPWL estimate
Timing estimate sơ bộ
Congestion risk estimate sơ bộ
```

Output này chưa đủ để handoff sang CTS/Routing.

---

## 5.11. [[HPWL]] — wirelength proxy trong Placement

[[HPWL]] là viết tắt của Half-Perimeter Wirelength.

Với một net có các pins tại nhiều tọa độ `(x, y)`, HPWL được tính bằng bounding box:

```text
HPWL(net) = (x_max - x_min) + (y_max - y_min)
```

Tổng HPWL của design thường là tổng HPWL của các nets được xét.

### Vì sao dùng HPWL?

HPWL được dùng vì:

- tính nhanh;
- đủ tốt để làm proxy sớm;
- phù hợp cho optimizer chạy nhiều vòng;
- cho biết xu hướng wirelength trước routing;
- hữu ích để so sánh placement solutions.

### HPWL bỏ qua gì?

HPWL không mô hình đầy đủ:

- routing blockage;
- macro obstruction;
- detour;
- layer assignment;
- via cost;
- congestion;
- pin access;
- real routing topology;
- DRC constraints;
- coupling/noise.

Vì vậy:

> HPWL là proxy, không phải wirelength thật sau Routing.

Một placement có HPWL thấp vẫn có thể route tệ nếu nó tạo congestion hoặc pin-access problem.

---

## 5.12. Timing-driven placement

Timing-driven placement dùng thông tin từ [[STA]], [[SDC]], [[MMMC]] và [[Slack]] để ưu tiên các timing-critical paths.

Mục tiêu:

- đặt cells trên critical paths gần nhau hơn;
- giảm estimated wire delay;
- giảm load/capacitance trên critical nets;
- cải thiện setup slack trước CTS;
- tránh để path quan trọng đi vòng qua macro/congestion region.

Ở pre-CTS context:

- clock thường còn ideal;
- propagated clock tree chưa tồn tại;
- clock skew thực chưa có;
- wire delay là estimate;
- setup timing thường là focus chính.

### Giới hạn của timing-driven placement

Timing-driven placement chưa thấy đầy đủ:

- clock tree thật;
- post-CTS skew;
- routed parasitics;
- coupling capacitance;
- final via/wire geometry;
- signoff extraction.

Vì vậy timing ở Placement là timing estimate, không phải signoff timing.

Tuy nhiên, nó vẫn rất quan trọng vì nếu timing đã xấu nặng ở post-placement, downstream closure sẽ khó.

---

## 5.13. Congestion-driven placement

Congestion-driven placement cố gắng phân bố cells sao cho routing demand không vượt routing supply cục bộ.

Các nguyên nhân congestion thường gặp:

- [[PlacementDensity]] cao cục bộ;
- pin density cao;
- macro channels hẹp;
- nhiều nets đi qua cùng vùng;
- [[PlacementBlockage]] đẩy cells sang vùng lân cận;
- [[RoutingBlockage]] giảm routing resource;
- [[PDN]] chiếm nhiều routing layers;
- macro obstructions làm route phải detour;
- IO/block pins cluster quá dày.

Congestion-driven placement có thể:

- spread cells;
- giảm local density;
- điều chỉnh vị trí cells;
- dùng congestion maps;
- phối hợp với [[DetailedPlacement]];
- gợi ý cần sửa floorplan nếu hotspot đến từ macro/floorplan topology.

### Trade-off

Spread cells giúp giảm congestion nhưng có thể làm wirelength tăng.

```text
Spread cells
  ↓
Local density giảm
  ↓
Congestion giảm
  ↓
Nhưng một số nets dài hơn
  ↓
Timing có thể xấu hơn
```

Vì vậy congestion-driven placement phải cân bằng với timing-driven placement.

---

## 5.14. [[PlacementDensity]]

[[PlacementDensity]] mô tả mức độ chiếm dụng area bởi standard cells trong legal placement area.

Một công thức cơ bản:

```text
Cell Density = Total Standard Cell Area / Total Placeable Area
```

Trong thực tế, cần phân biệt:

```text
Global utilization:
mức độ đặc trên toàn core / toàn placeable area

Local density:
mức độ đặc ở từng vùng nhỏ trong layout
```

Một design có global utilization có vẻ hợp lý vẫn có thể fail routing nếu local density quá cao.

Ví dụ:

```text
Global utilization = 65%
Nhưng quanh một macro, local density = rất cao
→ pin density cao
→ routing demand cao
→ congestion hotspot
```

### Density và blockage

[[PlacementBlockage]] ảnh hưởng trực tiếp đến placement density.

Nếu đặt nhiều blockage ở một vùng, cells bị đẩy sang vùng khác. Vùng khác có thể bị density cao lên và tạo congestion mới.

Vì vậy blockage strategy phải được kiểm tra bằng congestion analysis, không nên đặt cảm tính.

---

## 5.15. [[Legalization]]

[[Legalization]] là bước đưa kết quả từ [[GlobalPlacement]] về trạng thái placement hợp lệ.

Nó xử lý:

- cell overlap;
- cell lệch site;
- cell ngoài row;
- cell xâm phạm blockage;
- cell nằm sai legal region.

Một placement sau legalization cần:

- align đúng [[Site]];
- nằm trong [[Row]];
- không overlap;
- tôn trọng [[PlacementBlockage]];
- không xâm phạm fixed macros;
- giữ orientation hợp lệ.

### Movement vs legality trade-off

Legalization cần đạt legality với mức dịch chuyển nhỏ nhất có thể.

Nếu dịch chuyển quá mạnh:

- HPWL có thể tăng;
- timing posture có thể xấu;
- congestion map có thể thay đổi;
- placement solution từ Global Placement bị phá.

Nếu dịch chuyển quá ít:

- overlap hoặc legality violation còn tồn tại.

Vì vậy legalization không chỉ là “snap cell vào row”, mà là một bước tối ưu có ràng buộc.

---

## 5.16. [[DetailedPlacement]]

[[DetailedPlacement]] là bước tinh chỉnh cục bộ sau [[GlobalPlacement]] và [[Legalization]].

Mục tiêu:

- giảm local wirelength;
- cải thiện [[HPWL]];
- giữ placement legality;
- cải thiện local density;
- giảm congestion risk;
- cải thiện timing/congestion posture trước CTS/Routing.

Detailed Placement không được phá:

- site alignment;
- row legality;
- no-overlap;
- blockage constraints;
- fixed macro constraints.

### Giới hạn

Detailed Placement chỉ refine cục bộ. Nó không thể sửa triệt để:

- core quá chật;
- macro placement sai;
- PDN chiếm routing quá nhiều;
- floorplan tạo channel quá hẹp;
- timing architecture quá xấu.

Nếu lỗi là floorplan-level, phải quay lại [[Floorplanning]].

---

## 5.17. Early routability estimate và [[CongestionAnalysis]]

Sau Placement, cần kiểm tra routability trước khi đi tiếp.

[[CongestionAnalysis]] là hoạt động ước lượng routing pressure dựa trên:

- placement density;
- pin density;
- routing grid;
- routing blockages;
- macro obstruction;
- estimated routing demand;
- routing supply;
- early global routing nếu flow có.

Congestion xảy ra khi:

```text
routing demand > routing supply
```

ở một vùng cục bộ.

### Kết quả congestion dùng để làm gì?

Nếu congestion nhẹ:

- refine placement;
- spread cells;
- adjust density;
- thêm/bớt placement blockage;
- chạy lại detailed placement.

Nếu congestion nặng và lặp lại theo cấu trúc floorplan:

- quay lại [[Floorplanning]];
- chỉnh [[MacroPlacement]];
- mở routing channel;
- đổi blockage strategy;
- giảm utilization;
- chỉnh pin/macro orientation nếu cần.

Một câu cần nhớ:

> Congestion sau Placement là tín hiệu sớm để tránh fail ở Routing.

Không nên chờ đến [[DetailedRouting]] mới xử lý congestion do placement/floorplan gây ra.

---

## 5.18. [[HFNS]] — High Fanout Net Synthesis

[[HFNS]] xử lý các high-fanout nets trong Placement flow.

High-fanout nets thường là:

- reset;
- scan enable;
- test mode;
- clock gating enable;
- global control signals;
- non-clock nets drive rất nhiều sinks.

Nếu một net drive quá nhiều sinks:

- capacitance lớn;
- driver yếu;
- slew xấu;
- max transition violation;
- max capacitance violation;
- max fanout violation;
- timing estimate xấu;
- routing khó;
- congestion tăng.

HFNS thường chèn buffer tree để chia tải.

```text
One driver → quá nhiều sinks
  ↓
HFNS
  ↓
buffer tree
  ↓
mỗi buffer drive một nhóm sinks nhỏ hơn
```

### HFNS khác CTS như thế nào?

HFNS giống CTS ở ý tưởng buffer tree, nhưng khác đối tượng.

- [[HFNS]] xử lý non-clock high-fanout nets.
- [[ClockTreeSynthesis]] xử lý clock nets.

Không nên để CTS phải “giải quyết” các non-clock high-fanout nets. Những nets này nên được xử lý trước CTS nếu flow yêu cầu.

---

## 5.19. [[ScanChainReordering]]

[[ScanChainReordering]] sắp xếp lại thứ tự flip-flops trong scan chains dựa trên vị trí vật lý sau Placement.

Scan chains ban đầu thường được tạo bởi DFT flow khi chưa có thông tin placement. Do đó hai FF liền kề trong scan chain có thể nằm rất xa nhau.

Hậu quả nếu không reorder:

- scan nets dài;
- crisscross nhiều;
- routing congestion;
- wirelength tăng;
- test-related routing khó;
- PPA xấu hơn.

Sau Placement, tool biết vị trí physical của FFs. Scan chain reordering sắp xếp lại thứ tự trong chain để giảm wirelength giữa các scan elements.

### Điều cần giữ

Scan reordering thường cần giữ:

- chain count;
- chain balance;
- DFT correctness;
- scan connectivity hợp lệ;
- constraints của DFT/signoff flow.

Reordering thay đổi scan connectivity, nên database phải được update và DFT verification phải được kiểm soát theo flow.

---

## 5.20. [[DRVFixing]]

[[DRVFixing]] xử lý các electrical design-rule violations trong implementation.

Các DRV phổ biến trong pre-CTS / post-placement context:

- max capacitance;
- max transition;
- max fanout.

### max_capacitance

Constraint trên output pin.

Violation xảy ra khi driver phải drive load capacitance quá lớn.

Fix khái niệm:

- upsize driver;
- insert buffer;
- split net;
- reduce load nếu flow cho phép.

### max_transition

Constraint trên input pin.

Violation xảy ra khi input slew quá chậm.

Nguyên nhân thường là:

- driver yếu;
- load lớn;
- net dài;
- fanout cao;
- capacitance lớn.

Fix khái niệm:

- upsize driver;
- insert buffer;
- split long/high-capacitance net.

### max_fanout

Constraint trên output pin.

Violation xảy ra khi một driver feed quá nhiều sinks.

Fix khái niệm:

- insert buffers/repeaters;
- tạo buffer tree;
- HFNS nếu là high-fanout net lớn.

### Vì sao phải fix DRV trước CTS?

Nếu để DRV tồn tại:

- timing estimate kém ổn định;
- slew xấu làm cell delay phía sau xấu;
- CTS/Routing nhận một database không khỏe;
- downstream fix tốn kém hơn;
- power có thể tăng do short-circuit power khi slew xấu.

Một placed database có timing tạm ổn nhưng DRV nhiều vẫn chưa đủ tốt để handoff sang CTS.

---

## 5.21. [[PreCTSOptimization]]

[[PreCTSOptimization]] là giai đoạn tối ưu sau Placement và trước [[ClockTreeSynthesis]].

Bối cảnh pre-CTS:

- chưa có clock tree thật;
- clock thường còn ideal;
- clock sinks đã có vị trí;
- wire delay vẫn là estimate;
- setup timing là focus chính;
- hold timing thật thường rõ hơn sau CTS với propagated clock.

Mục tiêu:

- cải thiện setup timing;
- giảm WNS/TNS/FEP;
- fix DRV;
- giảm congestion risk;
- tối ưu power/area khi timing cho phép;
- chuẩn bị database đủ sạch cho CTS.

### Các kỹ thuật thường gặp

#### 1. Buffering

Chèn buffer/inverter để:

- chia tải;
- giảm slew;
- giảm delay trên long/high-cap nets;
- fix max transition;
- fix max capacitance;
- hỗ trợ setup timing.

Trade-off:

- area tăng;
- dynamic power tăng;
- placement density tăng;
- routing demand tăng.

#### 2. Cell sizing

Thay drive strength của cell.

Upsize:

- tăng drive;
- giảm delay;
- cải thiện slew;
- có thể fix setup/DRV;
- nhưng tăng area/power/cap input.

Downsize:

- giảm area/power;
- dùng khi path còn slack;
- nhưng có thể làm timing/slew xấu.

#### 3. VT swapping

Dùng cell có Vt khác nhau.

- LVT: nhanh hơn, thường leakage cao hơn.
- HVT: leakage thấp hơn, thường chậm hơn.
- SVT: trung gian.

Pre-CTS thường dùng LVT để cứu setup trên path critical, hoặc HVT để giảm leakage trên non-critical paths nếu slack cho phép.

Không được giả định mọi Vt variant luôn cùng footprint/pin equivalence nếu chưa xác nhận library.

#### 4. Logic restructuring

Thay đổi cấu trúc logic để cải thiện timing/area.

Ví dụ concept-level:

- decomposition;
- composition;
- xóa buffer/inverter dư;
- giảm logic depth;
- giảm cell count nếu phù hợp.

Đây là can thiệp mạnh hơn so với sizing/buffering.

#### 5. Cloning

Nhân bản logic để chia tải.

Ví dụ:

```text
Một driver feed nhiều sinks
  ↓
clone driver thành hai bản
  ↓
mỗi bản drive một nhóm sinks
```

Tác dụng:

- giảm fanout;
- giảm capacitance mỗi branch;
- cải thiện slew/timing.

Trade-off:

- area tăng;
- power có thể tăng;
- connectivity/database thay đổi.

#### 6. Pin swapping

Hoán đổi các input pins tương đương trên cùng một cell để cải thiện timing/power.

Ví dụ:

- critical net vào pin có delay thuận lợi hơn;
- high-toggle net vào pin có capacitance thấp hơn nếu hợp lệ.

Pin swapping phụ thuộc cell function và library timing arcs. Không phải pin nào cũng tương đương.

---

## 5.22. [[PowerAnalysis]] trong Placement

[[PowerAnalysis]] ở Placement giúp ước lượng power sớm.

Power thường được chia:

```text
Total Power = Static Power + Dynamic Power
```

Dynamic power gồm:

- switching power;
- internal power.

Switching power khái niệm:

```text
P_switching = α × C × V² × f
```

Trong đó:

- `α`: switching activity;
- `C`: capacitance;
- `V`: supply voltage;
- `f`: frequency.

Ở Placement stage, power analysis chưa chính xác như post-route vì:

- parasitics chưa thật;
- activity có thể chưa đầy đủ;
- routing capacitance chỉ là estimate;
- coupling chưa đầy đủ;
- final clock tree chưa có.

Nhưng nó vẫn hữu ích để:

- phát hiện vùng power cao;
- thấy xu hướng dynamic/leakage;
- hướng dẫn [[PowerOptimization]];
- tránh over-buffering;
- đánh giá tác động của cell sizing/Vt swap;
- dự báo IR drop risk sơ bộ.

---

## 5.23. [[PowerOptimization]] và [[LeakagePower]]

[[PowerOptimization]] thay đổi implementation để giảm power trong khi vẫn giữ timing/legal constraints.

Các lever thường gặp:

- cell sizing;
- Vt swap;
- drive-strength optimization;
- capacitance reduction;
- activity/cap-related optimization;
- downsizing non-critical paths;
- HVT swap trên paths có slack.

### Leakage Power

[[LeakagePower]] là static power, tồn tại ngay cả khi logic không switching.

Các yếu tố ảnh hưởng leakage:

- Vt;
- temperature;
- VDD;
- transistor width;
- process variation;
- logic state;
- device/process details.

Multi-Vt library là lever quan trọng:

```text
LVT → nhanh hơn, leakage thường cao hơn
HVT → leakage thấp hơn, chậm hơn
SVT → trung gian
```

### Trade-off

Không được hiểu PowerOptimization là “giảm power miễn phí”.

Ví dụ:

```text
Đổi LVT → HVT
  ↓
Leakage giảm
  ↓
Cell delay tăng
  ↓
Setup slack có thể xấu
```

Hoặc:

```text
Downsize cell
  ↓
Area/power giảm
  ↓
Drive yếu hơn
  ↓
Slew/DRV/timing có thể xấu
```

Power optimization phải bị ràng buộc bởi [[STA]], [[Slack]], DRV và placement legality.

---

## 5.24. [[MBFF]] — Multi-Bit Flip-Flop

[[MBFF]] là cell chứa nhiều flip-flop bits trong cùng một physical cell.

Ý tưởng:

```text
N single-bit FFs
  ↓ banking
1 multi-bit FF
```

Lợi ích tiềm năng:

- giảm số clock sinks;
- giảm clock capacitance;
- giảm dynamic power của clock network;
- giảm area trong một số trường hợp;
- cải thiện local skew/timing nếu grouping hợp lý.

Nhưng MBFF không phải lúc nào cũng tốt.

Rủi ro:

- giảm placement flexibility;
- cell lớn hơn;
- nếu merge FFs xa nhau, data path có thể dài hơn;
- timing constraints phải tương thích;
- clock domain phải tương thích;
- phụ thuộc library/tool support;
- không đảm bảo cải thiện PPA cho mọi design.

MBFF thường được cân nhắc ở post-placement / pre-CTS context, vì lúc này tool đã có thông tin locality vật lý.

Một câu cần nhớ:

> MBFF giảm clock sink count, nhưng có thể làm placement/data locality khó hơn nếu banking sai.

---

## 5.25. [[SpareCell]]

[[SpareCell]] là các standard-cell instances được pre-place trong layout để dự phòng ECO.

Chúng thường chưa tham gia active logic path tại thời điểm tape-out.

Mục tiêu:

- hỗ trợ ECO muộn;
- giảm nhu cầu re-placement lớn;
- cho phép metal-only ECO trong một số flow;
- cung cấp logic resource gần vùng cần sửa.

Spare cells thường được rải trong core.

Cách chọn:

- loại cell nào;
- mật độ bao nhiêu;
- đặt ở đâu;
- có FF hay chỉ combinational;
- tie-off như thế nào;

đều phụ thuộc project/flow/library.

### Tie-off

Input của SpareCell thường cần nối về known state thông qua [[TieCell]] để tránh floating input và switching không mong muốn.

Không nên để spare cells gây noise/power không cần thiết.

---

## 5.26. Placement và Tie Cells

[[TieCell]] không phải chủ đề trung tâm của Placement, nhưng nó liên quan đến physical implementation.

Tie cells dùng để tạo logic constant an toàn như tie-high hoặc tie-low.

Không nên nối trực tiếp gate input vào VDD/VSS nếu library/PDK không cho phép. Tie cells giúp:

- tạo constant logic;
- tránh stress reliability;
- kiểm soát fanout của constant nets;
- hỗ trợ spare cell input tie-off;
- hỗ trợ physical implementation đúng rule.

Cần phân biệt:

- [[TieCell]]: logic constant.
- [[TapCell]]: well/substrate bias.
- [[FillerCell]]: row continuity.
- [[DecapCell]]: local decoupling.
- [[EndCapCell]]: row-end termination.

---

## 5.27. Placement qualification trước CTS

Placement không nên handoff sang CTS chỉ vì cells đã legal.

Cần kiểm tra:

```text
Legality clean
No overlap
Rows/sites respected
Macro blockages respected
Utilization hợp lý
Local density không quá xấu
Congestion estimate chấp nhận được
Pre-route setup timing feasible
WNS/TNS/FEP trong mức flow chấp nhận
DRV violations đã được xử lý ở mức hợp lý
HFNS đã xử lý high-fanout nets quan trọng
Scan chain reordering đã xử lý nếu design có scan
Power estimate không có dấu hiệu bất thường
MBFF/spare/tie-related cleanup hợp lý nếu flow dùng
Placed DB / DEF ready for CTS
```

Nếu placement có vấn đề nghiêm trọng, cần sửa trước CTS.

Lý do:

> CTS sẽ xây clock tree dựa trên vị trí cells hiện tại. Nếu placement xấu, CTS sẽ hợp thức hóa một topology xấu và làm chi phí sửa tăng lên.

---

## 5.28. Placement ảnh hưởng CTS như thế nào?

[[ClockTreeSynthesis]] cần clock sink locations.

Sau Placement:

- mọi flip-flop đã có tọa độ;
- clock pins có vị trí;
- macro clock pins đã cố định;
- register clusters đã hình thành;
- clock load distribution đã rõ hơn.

Placement xấu có thể gây:

- clock sinks phân tán quá rộng;
- clock latency cao;
- skew khó cân bằng;
- clock buffers phải chèn nhiều;
- clock power tăng;
- hold/setup risk thay đổi sau CTS.

Đặc biệt, nếu nhiều sequential cells trên cùng clock domain bị đặt rải rác, CTS phải route clock dài hơn và cân bằng khó hơn.

---

## 5.29. Placement ảnh hưởng Routing như thế nào?

Routing nhận placed cells và fixed pins.

Placement quyết định:

- pin density;
- net length;
- routing demand;
- local congestion;
- macro-channel demand;
- need for detour;
- pin access difficulty;
- cell density near blockages;
- available routing resources.

Một placement legal nhưng routability kém sẽ gây:

- global routing congestion;
- detailed routing DRC khó clean;
- detours;
- via count tăng;
- wirelength tăng;
- [[NetDelay]] tăng;
- timing degradation;
- possibly routing failure.

Một câu cần nhớ:

> Routing không thể tạo tài nguyên routing từ hư không. Nếu Placement nhồi quá nhiều pins vào một vùng thiếu tracks, router sẽ phải detour hoặc fail.

---

## 5.30. Placement ảnh hưởng STA như thế nào?

Sau Placement, STA có thể dùng estimated wire delay dựa trên cell locations.

So với zero-RC timing ở Design Import:

- wire delay estimate thực tế hơn;
- cell-to-cell distance đã có;
- fanout/load estimate tốt hơn;
- setup timing feasibility rõ hơn.

Nhưng vẫn chưa có:

- clock tree thật;
- routed wires thật;
- extracted RC thật;
- coupling capacitance thật.

Vì vậy post-placement STA là checkpoint trung gian.

Nó trả lời:

```text
Với placement hiện tại và wire delay estimate, design có timing posture đủ tốt để đi CTS không?
```

Không phải:

```text
Design đã signoff timing clean chưa?
```

---

## 5.31. Khi nào phải quay lại Floorplanning?

Không phải mọi lỗi sau Placement đều sửa trong Placement.

Cần quay lại [[Floorplanning]] nếu:

- congestion hotspot lặp lại do macro channel quá hẹp;
- standard-cell area bị chia nát;
- utilization quá cao;
- macro placement sai hướng dataflow;
- macro pins quay vào vùng không route được;
- PDN chiếm quá nhiều routing resource;
- blockages làm placeable area quá nhỏ;
- boundary/aspect ratio gây routing imbalance lớn;
- IO/block pin distribution quá xấu.

Nguyên tắc:

> Nếu lỗi đến từ search space, sửa Placement không đủ. Phải sửa Floorplanning.

Placement chỉ tối ưu bên trong không gian mà Floorplanning cho phép.

---

## 5.32. Những lỗi Placement phổ biến của người mới

### Lỗi 1 — Xem Placement là “đặt cell lên row”

Sai:

> Placement chỉ là assign tọa độ cho standard cells.

Đúng:

> Placement là pre-CTS implementation package gồm global placement, legalization, detailed placement, timing/congestion optimization, DRV cleanup, HFNS, scan reorder, power/area optimization và placement qualification.

---

### Lỗi 2 — Chỉ nhìn HPWL

Sai:

> HPWL thấp nghĩa là placement tốt.

Đúng:

> HPWL chỉ là proxy. Placement còn phải kiểm tra timing, density, congestion, pin access, legality, power và downstream routability.

---

### Lỗi 3 — Nhầm Global Placement với legal placement

Sai:

> Global Placement xong là cells đã hợp lệ.

Đúng:

> [[GlobalPlacement]] có thể còn overlap hoặc lệch site. Cần [[Legalization]] và [[DetailedPlacement]] để tạo legal placement.

---

### Lỗi 4 — Chỉ nhìn global utilization

Sai:

> Global utilization ổn thì placement ổn.

Đúng:

> Local density mới là vấn đề routing. Một vùng nhỏ quá đặc vẫn có thể tạo congestion hotspot dù global utilization đẹp.

---

### Lỗi 5 — Để DRV sang sau CTS

Sai:

> DRV để CTS/Routing sửa sau cũng được.

Đúng:

> DRV tồn tại trước CTS làm timing/power/routing kém ổn định. Nên cleanup ở pre-CTS nếu flow yêu cầu.

---

### Lỗi 6 — Dùng buffer quá nhiều để sửa timing

Sai:

> Thêm buffer là cách fix timing đơn giản, cứ thêm nhiều là tốt.

Đúng:

> Buffering tăng area, power, density và routing demand. Over-buffering có thể tạo congestion và power problem.

---

### Lỗi 7 — Merge MBFF không xét physical locality

Sai:

> Cứ gom FF thành MBFF là giảm power.

Đúng:

> MBFF chỉ tốt khi timing, clock domain, library support và physical locality phù hợp. Merge sai có thể làm data path dài hơn và placement kém linh hoạt.

---

### Lỗi 8 — Không reorder scan chain

Sai:

> Scan chain là DFT, không liên quan placement.

Đúng:

> Scan chain ban đầu có thể placement-unaware. Sau placement, scan reorder giúp giảm wirelength và congestion cho scan nets.

---

### Lỗi 9 — Tin Placement pass chỉ vì no-overlap

Sai:

> Cells không overlap nghĩa là placement tốt.

Đúng:

> No-overlap chỉ là legality tối thiểu. Placement còn phải pass timing, congestion, DRV, density, power và downstream readiness.

---

## 5.33. Checklist học tập cho chương 5

Sau chương này, người học nên trả lời được:

1. [[Placement]] nhận input gì từ [[Floorplanning]]?
2. Vì sao Placement không chỉ là đặt cells lên [[Row]]?
3. [[GlobalPlacement]] khác [[DetailedPlacement]] như thế nào?
4. Vì sao [[GlobalPlacement]] có thể cho overlap tạm thời?
5. [[Legalization]] giải quyết vấn đề gì?
6. [[HPWL]] là gì và nó bỏ qua những yếu tố nào?
7. Vì sao HPWL thấp chưa chắc routing tốt?
8. [[PlacementDensity]] khác global utilization như thế nào?
9. Vì sao local density quan trọng hơn trong congestion?
10. [[CongestionAnalysis]] sau Placement dùng để quyết định gì?
11. Khi nào congestion phải quay lại [[Floorplanning]] thay vì sửa Placement?
12. [[HFNS]] xử lý loại nets nào?
13. [[HFNS]] khác [[ClockTreeSynthesis]] như thế nào?
14. [[ScanChainReordering]] dùng để giảm vấn đề gì?
15. [[DRVFixing]] gồm những loại violation nào?
16. max_capacitance, max_transition và max_fanout khác nhau ở đâu?
17. [[PreCTSOptimization]] tối ưu gì trước CTS?
18. Buffering, sizing, VT swapping, cloning, pin swapping khác nhau thế nào?
19. [[PowerAnalysis]] ở Placement có giới hạn gì?
20. [[PowerOptimization]] có thể làm timing xấu hơn như thế nào?
21. [[MBFF]] có lợi ích và rủi ro gì?
22. [[SpareCell]] dùng cho mục tiêu gì?
23. Placement ảnh hưởng [[ClockTreeSynthesis]] như thế nào?
24. Placement ảnh hưởng [[Routing]] như thế nào?
25. Khi nào placed database đủ điều kiện handoff sang CTS?

---

## 5.34. Quan hệ với các chương trước và sau

Chương 1 đã giới thiệu input của Physical Design:

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
[[Site]] → [[Row]] → [[PlacementGrid]]
[[Pitch]] → [[Track]] → [[RoutingGrid]]
[[CellAbstract]] → [[Pin]] + [[Obstruction]]
```

Chương 3 đã tạo validated design database:

```text
[[DesignImport]]
  ↓
Validated design database
```

Chương 4 đã tạo physical search space:

```text
[[Floorplanning]]
  ↓
Core boundary + macros + rows + blockages + PDN
```

Chương 5 dùng search space đó để đặt standard cells:

```text
[[Placement]]
  ↓
Placed standard cells + pre-CTS optimized database
```

Chương tiếp theo là:

```text
# Chương 6 — Clock Tree Synthesis
```

Ở chương 6, người học sẽ học cách [[ClockTreeSynthesis]] biến ideal clock thành physical clock tree, chèn clock buffers/inverters, kiểm soát [[ClockSkew]], [[ClockLatency]], clock slew, clock power, và chuẩn bị design cho [[Routing]].

---

## 5.35. Tóm tắt chương 5

[[Placement]] là giai đoạn pre-CTS implementation package, không chỉ là thao tác đặt cell.

Các lớp chính:

```text
Pre-placement sanity
[[GlobalPlacement]]
[[Legalization]]
[[DetailedPlacement]]
[[HFNS]]
[[ScanChainReordering]]
[[DRVFixing]]
[[PreCTSOptimization]]
[[PowerAnalysis]]
[[PowerOptimization]]
[[MBFF]]
[[SpareCell]]
[[CongestionAnalysis]]
Placement qualification
```

Các objective chính:

```text
Legality
Wirelength
Timing
Density
Congestion
DRV cleanliness
Power/area
CTS readiness
Routing readiness
```

Output chính:

```text
Placed database
Legalized cell coordinates
Placed [[DEF]]
Timing/congestion/power/DRV reports
Clock sink locations
Handoff-ready database for [[ClockTreeSynthesis]]
```

Câu cần nhớ:

> Placement tốt không chỉ là cell không overlap. Placement tốt là placed database đủ legal, timing-feasible, routable, electrically clean và sẵn sàng để CTS xây clock tree trên đó.