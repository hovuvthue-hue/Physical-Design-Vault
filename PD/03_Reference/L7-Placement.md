# Lecture 7 – Placement (Part I): Phân Tích Lý Thuyết Toàn Diện

---

## 1. Vị Trí Của Placement Trong Physical Design Flow

Placement là bước thứ ba trong physical design flow, nằm sau Floorplan và trước CTS (Clock Tree Synthesis):

**Data Import → Floorplan → Placement → CTS → Routing → Data Export**

Placement không đơn thuần là gán vị trí vật lý cho các Standard Cell — trong giai đoạn này, Netlist còn được tối ưu hóa để đạt PPA (Power, Performance, Area) tốt hơn. Đây là điểm then chốt: Placement và Optimization diễn ra đồng thời, không tách rời.

---

## 2. Các Khái Niệm Nền Tảng

### 2.1 Standard Cell là gì?

Một Standard Cell là một logic cell đã được pre-characterized (đặc trưng hóa trước) và well-defined, dùng làm building block cơ bản trong thiết kế. Các thuộc tính quan trọng:

- **Chiều cao đồng nhất (uniform height):** tất cả Standard Cell trong cùng một thư viện có chiều cao bằng nhau, cho phép chúng xếp thành hàng (rows) liên tục.
- **Chiều rộng thay đổi (variable width):** phụ thuộc vào chức năng logic của cell (ví dụ: Inverter nhỏ hơn Latch).
- **Đa dạng về chiều cao:** có thể có single-height cell và double-height cell. Double-height cell chiếm 2 rows liền kề, thường dùng cho các cell phức tạp hoặc cần drive strength cao hơn.

Ý nghĩa vật lý: single-height cell chỉ chiếm một row, double-height cell chiếm hai rows liền kề — điều này ảnh hưởng trực tiếp đến cách tool thực hiện Placement và legalization.

### 2.2 Site là gì?

Một **Site** định nghĩa đơn vị lưới tối thiểu (minimum grid unit) gồm width và height, mà trên đó một Standard Cell có thể được đặt. Các thuộc tính:

- Standard Cell phải được căn chỉnh (align) vào site grid bên trong các placement rows.
- Các cell lớn hơn có thể chiếm nhiều sites theo chiều ngang và/hoặc chiều dọc.
- Site được định nghĩa trong technology library, ví dụ: `SITE CORE; CLASS CORE; SYMMETRY X Y; SIZE 0.2 X 12.0; END CORE`.

Site là nền tảng của tính hợp lệ (legality) trong Placement: một cell chỉ được coi là "legally placed" khi nó snap vào đúng site grid.

### 2.3 Row là gì?

Một **Row** là cấu trúc placement được định nghĩa bởi technology library, là dải ngang (horizontal band) chứa các Standard Cell trong quá trình Placement. Các thuộc tính:

- Chiều cao cố định, nhất quán với chiều cao của Standard Cell mà nó chứa.
- Một Row được tạo thành từ chuỗi các site liền kề.
- Các design tiên tiến (lower geometry) có thể có multiple row heights.
- Rows có thể có các orientation khác nhau: **R0**, **MX**, flipped/abut configurations.

**Mối quan hệ Site–Row–Cell:** một Row = nhiều Sites liền kề theo chiều ngang. Một Cell khi đặt vào Row phải occupy một số nguyên các Sites theo chiều rộng.

### 2.4 Cell Orientation trong Row

Row orientation xen kẽ giữa **R0** và **MY** để kích hoạt cell flipping:

- **R0 (upright):** Cell đứng thẳng, không xoay, không mirror. Power rail VDD ở trên, VSS ở dưới.
- **MY (Mirror Y-axis / flip top-to-bottom):** Cell được lật theo trục X (đảo ngược chiều dọc). Power rail VSS ở trên, VDD ở dưới — điều này cho phép row bên trên và row bên dưới chia sẻ cùng một power rail.

Bốn orientation hợp lệ của Standard Cell:

|Orientation|Ý nghĩa|
|---|---|
|**R0**|Default, không xoay, không mirror|
|**MY**|Mirror theo trục Y (flip top-to-bottom)|
|**MX**|Mirror theo trục X (flip left-to-right)|
|**R180**|Xoay 180°|

**Tại sao cần orientation xen kẽ?** Khi hai row liền kề có orientation R0 và MY, power rail của chúng có thể **abutted** (tiếp giáp trực tiếp): VDD của row R0 chạm VDD của row MY, VSS của row R0 chạm VSS của row MY — điều này cho phép chia sẻ power rail và giảm số lượng dây nguồn, tiết kiệm diện tích và giảm routing complexity.

**Double-back rows:** Hai rows lồng vào nhau, cả VDD lẫn VSS đều được kết nối chung, tạo ra cấu trúc abutted N-Well giữa các rows liền kề.

### 2.5 Standard Cell Placement Area

- Tool chỉ đặt Standard Cell **bên trong** core area (khu vực được xác định trong Floorplan).
- Cell sẽ không bao giờ được đặt ngoài core area.
- Cell Pin phải nằm tại giao điểm của cả horizontal và vertical routing grid.
- Cell Height phải là bội số nguyên của horizontal routing grid.
- Cell Width phải là bội số nguyên của vertical routing grid.

---

## 3. Mục Tiêu Của Placement

Placement hướng đến năm mục tiêu chính, có thứ tự ưu tiên:

**1. Đạt được timing và power goals** Setup timing và Hold timing phải được đáp ứng; power phải được minimize. Đây là mục tiêu PPA cốt lõi.

**2. Minimize congestion** Phân bổ cell đều để tránh routing bottleneck. Congestion cao → routing khó hoặc impossible → design fail.

**3. Control placement density** Duy trì density tối ưu để để lại không gian cho timing optimization và routing về sau. Density quá cao → không có chỗ insert buffer → timing không converge.

**4. Tránh Design Rule Violations (DRVs)** Tuân thủ cell spacing, max transition, max capacitance, max fanout constraints.

**5. Enable a routable design** Placement phải setup layout sao cho CTS và Routing có thể thực hiện thành công ở các bước tiếp theo.

---

## 4. Key Inputs To Placement

Trước khi chạy Placement, các inputs sau phải sẵn sàng:

- **Netlist:** Mô tả kết nối logic giữa các cell (định dạng Verilog gate-level).
- **Physical Libraries (LEF):** Chứa thông tin vật lý của cell (kích thước, pin locations, metal layers). LEF = Library Exchange Format.
- **Floorplan (DEF):** Chứa thông tin về die/core boundary, IO pad locations, macro placements, placement rows, routing tracks. DEF = Design Exchange Format.
- **MCMM Setup (Multi-Corner Multi-Mode):**
    - **Timing Libraries (.lib):** Chứa timing characterization của cell (delay, setup time, hold time, v.v.) ở các corners khác nhau (slow/fast, hot/cold).
    - **Timing Constraints (SDC):** Chứa clock definitions, input/output delays, false paths, multicycle paths. SDC = Synopsys Design Constraints.
    - **RC Corners:** Mô tả resistance và capacitance của interconnect ở các process corners.
- **Power Constraints (UPF):** Relevant cho multi-voltage designs. UPF = Unified Power Format — định nghĩa power domains, level shifters, isolation cells.

Tất cả inputs này hợp thành **Floorplan Database** — trạng thái thiết kế cuối cùng sau Floorplan, sẵn sàng cho Placement.

---

## 5. Placement Density

**Placement Density** là tỷ lệ diện tích đã được chiếm bởi Standard Cell so với tổng diện tích có thể đặt cell (placeable area).

Có hai metric:

$$\text{Cell Density} = \frac{\text{Total Standard Cell Area}}{\text{Total Placeable Area}}$$

$$\text{Core Density} = \frac{\text{Total Standard Cell Area} + \text{Total Macro Area}}{\text{Total Placeable Area}}$$

**Guideline thực tế:** Placement density thường được giữ dưới **~70%** để đảm bảo routability tốt. Lý do:

- Cần không gian để insert Buffer/Inverter trong quá trình timing optimization (preCTS và postCTS).
- Cần routing resource cho metal tracks.
- Higher density → increased congestion → routing failures.

---

## 6. Placement Blockage

Placement Blockage là các vùng mà tool bị hạn chế hoặc cấm đặt cell, dùng để kiểm soát phân bố cell, giảm congestion, và bảo vệ các vùng critical.

|Loại|Mô tả|
|---|---|
|**Hard Blockage**|Hoàn toàn cấm đặt cell. Không có Standard Cell nào được phép trong vùng này.|
|**Partial Blockage**|Cho phép đặt cell nhưng với density bị giới hạn đến một tỷ lệ phần trăm nhất định.|
|**Soft Blockage**|Chủ yếu cấm regular cell; chỉ Buffer/Inverter (dùng cho optimization) được phép, với density tối đa xác định.|

---

## 7. Routing Tracks và Metal Layers

### 7.1 Metal Layers

Design hiện đại dùng từ 3 đến 19 metal layers để routing. Các metal layers liền kề được kết nối qua via. Mỗi metal layer có **preferred routing direction** (chiều routing ưu tiên):

- Các layer xen kẽ nhau giữa horizontal và vertical.
- Ví dụ: M1 (horizontal) → M2 (vertical) → M3 (horizontal) → v.v.

Cấu trúc xen kẽ này giảm crosstalk, cải thiện routing efficiency, và đảm bảo manufacturability tốt hơn.

### 7.2 Routing Tracks

**Routing Track** là các đường dẫn được định nghĩa sẵn mà router dùng để kết nối Net giữa các cell. Các thuộc tính:

- Tracks được sắp xếp theo chiều ngang và chiều dọc trên các routing layers.
- **Pitch:** Khoảng cách giữa hai routing tracks liền kề.
- Tracks tạo nên một structured grid hướng dẫn routing và đảm bảo tuân thủ design rules.

**Mối quan hệ Row–Track:** Track height của row (ví dụ: 7-track, 9-track) xác định có bao nhiêu routing tracks nằm trong chiều cao một row. Row với track number lớn hơn → nhiều routing resource hơn → cell phức tạp hơn có thể được đặt.

---

## 8. Placement Flow Overview

Toàn bộ Placement Flow được chia thành ba giai đoạn chính, có thể lặp lại (iterate):

```
Pre-Placement → Placement → Post-Placement
     ↑__________________________|
              ITERATIONS
```

**Pre-Placement:** Sanity checks

**Placement:**

- Global placement
- Detailed placement
- High Fanout Net Synthesis (HFNS)
- Scan chain reordering

**Post-Placement:**

- Design Rule Violation (DRV) fixing
- PreCTS timing optimization
- Power and Area optimizations
- Tie-cells insertion
- Add spare cells
- MBFF optimization
- Congestion analysis
- Guides to minimize congestion

---

## 9. Pre-Placement: Sanity Checks

Trước khi chạy Placement, cần xác minh rằng Floorplan database đã sẵn sàng và hợp lệ. Các điều kiện phải đạt:

1. **Tất cả I/O ports và pins** đã được đặt đúng vị trí chỉ định và được **fixed** (không di chuyển trong quá trình Placement).
2. **Tất cả Macros** đã được placed đúng và **locked** (cố định).
3. **Tất cả Standard Cell rows** đã được tạo và căn chỉnh đúng theo technology grid.
4. **Macro halos** và **Placement Blockages** đã được defined đúng. Macro halo là vùng exclusion zone xung quanh macro, ngăn cell đặt quá gần macro (để tránh routing congestion xung quanh macro boundary).
5. **Power and Ground (PG) nets** đã được kết nối và không có shorts.

---

## 10. Placement Strategies

Tool có thể optimize Placement theo nhiều chiến lược, tương ứng với các mục tiêu khác nhau:

|Objective|Strategy|Cơ chế|
|---|---|---|
|Area, Wirelength, Overlap|**Core Placement**|Minimize tổng wirelength, loại bỏ overlap|
|Timing|**Timing-Driven Placement**|Reduce wirelength để meet timing|
|Congestion|**Congestion-Driven Placement**|Spread out cells để reduce congestion|
|Clock|**Clock Gating**|Reduce clock switching và activity factor|
|Power|**Power Optimization**|Reduce power bằng cách chọn low-power cells|

Các chiến lược này không loại trừ nhau; tool sẽ cân bằng (trade-off) giữa chúng dựa trên cấu hình effort level.

---

## 11. Global Placement

**Global Placement** là giai đoạn placement thô (coarse), trong đó Standard Cell được phân bổ trên toàn bộ core area. Đặc điểm:

- Tập trung optimize tổng wirelength và congestion (early PPA improvement).
- Cells **chưa được căn chỉnh** vào placement rows ở giai đoạn này.
- **Overlap giữa các cells được phép** — đây là trạng thái trung gian, sẽ được resolve ở Detailed Placement.
- Tạo ra approximate placement làm input cho legalization và Detailed Placement.

**Nguyên lý hoạt động:** Tool sử dụng kỹ thuật analytical hoặc force-directed để minimize một objective function (thường là tổng half-perimeter wirelength — HPWL). Cells được coi như các "lực" đẩy/kéo nhau dựa trên net connectivity. Cells kết nối nhiều net với nhau sẽ bị kéo gần nhau hơn.

### 11.1 Global Routing (Early Routing Estimation)

Song song với Global Placement, tool thực hiện **Global Routing** ở mức độ coarse để ước tính routing feasibility. Cơ chế:

- Chia design thành lưới các **GCELLs** (global routing cells) — mỗi GCELL đại diện cho một vùng của chip chứa một số lượng nhất định routing tracks.
- Ước tính wirelength và routing demand trong từng GCELL.
- Xác định congestion hotspots tiềm năng (vùng mà routing demand vượt quá supply).
- Gán approximate routing resource (horizontal và vertical) cho từng GCELL.

**Lưu ý quan trọng:** Cells chưa được routed thực sự — đây là predictive routing step. Kết quả được đánh dấu status "unknown". Tool dùng thông tin này để hướng dẫn Global Placement đưa ra decisions tốt hơn.

**Metric chính: Overflow (%)**

$$\text{Overflow} = \frac{\text{Routing Demand} - \text{Routing Supply}}{\text{Routing Supply}} \times 100\%$$

- Overflow < 1% → generally routable
- Overflow > 1% → likely routing challenges (cần adjust Placement)

---

## 12. Detailed Placement

**Detailed Placement** tinh chỉnh vị trí cell từ kết quả Global Placement, đảm bảo tính hợp lệ pháp lý. Các đặc điểm:

- Cells được căn chỉnh vào placement rows và legal sites.
- **Legalization:** Đảm bảo không có overlap giữa các cells (0 cell overlaps, 0 site violations).
- Tinh chỉnh vị trí cell dựa trên: configured placement strategies, timing requirements, net connectivity, wirelength optimization, congestion awareness.
- Cells kết nối nhiều net với nhau được đặt gần nhau hơn → giảm delay, cải thiện PPA.
- Tạo ra **legal và optimized placement** sẵn sàng cho CTS và Routing.

**Legalization shift:** Khi legalize, cells có thể bị dịch chuyển (shift) từ vị trí global placement đến vị trí legal gần nhất. Shift nhỏ → placement tốt. Shift lớn → global placement không tốt, có thể cần iteration.

Ví dụ kết quả sau Detailed Placement:

- Standard Cells Placed: 1,203,488
- Total Placement Rows: 1,150
- Row Utilization: 68.2%
- Whitespace Remaining: 31.8%
- 0 cell overlaps, 0 site violations
- Average legalization shift: 0.89 µm; Maximum shift: 5.12 µm
- WNS: cải thiện từ -212ps lên -148ps
- TNS: cải thiện từ -49ns lên -28ns
- Congestion hotspots: giảm từ 7 xuống 3

---

## 13. High Fanout Net Synthesis (HFNS)

**HFNS** là bước tối ưu hóa để xử lý các Net có fanout cao (ví dụ: reset, scan enable, control signals). Vấn đề cốt lõi:

**Vấn đề:** Một driver cell duy nhất không thể drive quá nhiều load (fanout quá lớn) vì:

- Capacitance tổng cộng quá lớn → transition time (slew) chậm → max_transition violation.
- Delay tăng → setup timing violation.
- Overloading driver cell → signal integrity kém.

**Giải pháp của HFNS:**

1. Tool xác định các Net có fanout vượt quá ngưỡng cho phép (max_fanout constraint).
2. Insert Buffers để split load và phân phối signal theo dạng tree (buffer tree).
3. Cân bằng capacitance và cải thiện signal integrity.

**Lợi ích:**

- Giảm delay và transition violations.
- Cải thiện timing và signal reliability.
- Ngăn overloading của driver cells.

**Nguyên lý buffer tree:** Thay vì một driver → N loads, tạo ra cấu trúc: một driver → k buffers → mỗi buffer drive N/k loads. Có thể có nhiều tầng (tree depth = 2, 3, v.v.).

---

## 14. Scan Chain Reordering

### 14.1 Scan Chain là gì?

**Scan Chain** là một chuỗi các Flip-Flop được kết nối nối tiếp nhau cho mục đích DFT (Design For Test). Cơ chế:

- Dùng để shift test data vào và ra khỏi chip.
- Cho phép detect các manufacturing defects bằng cách control và observe trạng thái của từng Flip-Flop.
- DFT thường tạo ra nhiều scan chains có độ dài bằng nhau.

### 14.2 Tại sao cần Reordering?

Scan chains ban đầu (từ DFT tool) được tạo ra **không aware về physical placement** — thứ tự Flip-Flop trong chain không phản ánh vị trí vật lý của chúng. Hậu quả:

- Wirelength giữa các scan elements rất dài.
- Routing congestion tăng cao.
- Tốn nhiều metal resource.

### 14.3 Scan Chain Reordering trong Placement

Tool sắp xếp lại thứ tự Flip-Flop trong scan chain dựa trên **vị trí vật lý thực tế** của chúng sau Placement:

- Minimize tổng wirelength giữa các scan elements.
- Cải thiện routing và giảm congestion.
- Preserves chain balance (giữ nguyên số lượng Flip-Flop trong mỗi chain).
- Scan chain vẫn hoạt động đúng về mặt DFT — chỉ thứ tự kết nối thay đổi, không phải chức năng.

**Lợi ích tổng thể:** Faster test time, better routability, improved PPA.

---

## 15. Post-Placement: DRV Fixing

Sau khi Detailed Placement và HFNS hoàn thành, tool fix các **Design Rule Violations (DRV)** — vi phạm các electrical constraints:

### 15.1 Max Capacitance (max_cap)

**Định nghĩa:** Giá trị capacitance tải (load capacitance) tối đa mà một output pin được phép drive. Được định nghĩa trong cell library hoặc bởi designer.

**Vi phạm xảy ra khi:** Net quá dài, hoặc driver phải drive quá nhiều load → tổng capacitance vượt quá max_cap.

**Fix methods:**

- Tăng drive strength của driver (cell sizing lên larger cell).
- Insert Buffers dọc theo Net dài hoặc Net có nhiều load.
- Net splitting hoặc hierarchical refactoring nếu buffering không đủ.

### 15.2 Max Transition (max_transition)

**Định nghĩa:** Thời gian chuyển tiếp tín hiệu (transition time / slew) tối đa được phép tại một input pin. Áp dụng cho **input pins** của cells, không phải output pins.

**Vi phạm xảy ra khi:** Driver quá yếu hoặc Net có capacitance quá lớn → signal rise/fall time chậm → vượt max_transition.

**Fix methods:**

- Tăng drive strength của driver.
- Insert Buffers để split Net dài/high-capacitance và kiểm soát slew.

### 15.3 Max Fanout (max_fanout)

**Định nghĩa:** Số lượng load tối đa mà một output pin có thể drive. Được định nghĩa trong library hoặc bởi designer.

**Vi phạm xảy ra khi:** Một driver feed quá nhiều gates → excessive capacitance → slower transitions.

**Fix methods:**

- Insert Buffers/repeaters để split fanout load và restore signal strength.
- Logic restructuring: tree-based duplication cho clock hoặc enable signals.

---

## 16. Pre-CTS Timing Optimization

### 16.1 Nguyên lý Timing Closure

Timing optimization diễn ra ở nhiều giai đoạn trong flow, độ chính xác tăng dần:

|Stage|Clock model|Timing checks|RC Accuracy|
|---|---|---|---|
|Placement|Ideal clock (zero delay)|Setup only|Wire estimation|
|CTS|Real clock tree|Setup + Hold|Clock net routed|
|Route|Real clock + detail route|Setup + Hold|Full RC extraction|
|STA|Final database|Setup + Hold|Full Layout RCs|

Lý do Placement chỉ optimize Setup: chưa có clock tree thực → không biết insertion delay → không thể tính clock skew thực → Hold timing không có ý nghĩa ở giai đoạn này.

### 16.2 Setup Timing Check

**Timing path cơ bản (Register-to-Register):**

$$\text{Arrival Time} = T_{ck2q} + T_{combo}$$

$$\text{Required Time} = T_{clock_period} - T_{setup}$$

$$\boxed{\text{Setup Slack} = \text{Required Time} - \text{Arrival Time} = T_{period} - T_{setup} - T_{ck2q} - T_{combo}}$$

Trong đó:

- $T_{ck2q}$: Clock-to-Q delay của Flip-Flop launch
- $T_{combo}$: Tổng combinational logic delay trên data path
- $T_{setup}$: Setup time requirement của Flip-Flop capture
- $T_{period}$: Clock period

**Điều kiện meet timing:** Setup Slack ≥ 0

**Với clock skew:**

$$\text{Setup Slack} = T_{period} - T_{setup} - T_{ck2q} - T_{combo} + (T_{capture} - T_{launch})$$

Trong đó $(T_{capture} - T_{launch})$ là clock skew. Ở Pre-CTS, clock skew = 0 (ideal clock).

### 16.3 Các loại Timing Path

**4 loại path chính:**

1. **Register-to-Register (reg2reg):** Flip-Flop → combinational logic → Flip-Flop. Đây là path phổ biến nhất và quan trọng nhất.
    
    - Setup: $clk \to Q + combo \leq T_{period} - T_{setup}$
    - Hold: $clk \to Q + combo \geq T_{hold} + T_{reg_hold}$
2. **Input-to-Register (in2reg / input):** Primary input port → Flip-Flop. Cần set input delay constraint.
    
    - Setup Requirement: $T_{input_delay} + T_{combo2} \leq T_{period} - T_{reg_setup}$
    - Hold Requirement: $T_{input_delay} + T_{combo2} \geq T_{hold_check} + T_{reg_hold}$
3. **Register-to-Output (reg2out / output):** Flip-Flop → Primary output port. Cần set output external delay.
    
    - Setup: $clk \to Q + T_{combo1} \leq T_{period} - T_{external_delay}$
    - Hold: $clk \to Q + T_{combo1} \geq T_{hold_check} - T_{external_delay}$
4. **Input-to-Output (in2out):** Primary input → Primary output (purely combinational path).
    

**Các path group bổ sung:**

- **in2icg:** Input → ICG (Integrated Clock Gating) cell
- **reg2icg:** Register → ICG cell
- **reg2mem:** Register → Memory (black box)
- **mem2reg:** Memory → Register

### 16.4 Timing Terminology

|Term|Ý nghĩa|Quan hệ với timing|
|---|---|---|
|**WNS (Worst Negative Slack)**|Slack âm lớn nhất (worst violation)|Negative → có violation; mục tiêu → 0|
|**TNS (Total Negative Slack)**|Tổng tất cả slack âm|Large negative → widespread issues|
|**FEP (Failing Endpoints)**|Số endpoints vi phạm timing|High → nhiều paths cần fix|

**Goal của Pre-CTS Optimization:**

- Reduce WNS → 0 (fix worst path)
- Minimize TNS và FEP
- Prepare design cho clock-aware optimization trong CTS

### 16.5 On-Chip Variation (OCV)

**OCV** là hiện tượng process variation xảy ra trong quá trình fabrication, gây ra sự khác biệt về tham số transistor (length, width, oxide thickness):

- **Chip-to-chip variation:** Giữa các dies/wafers/lots khác nhau.
- **On-chip variation:** Trong cùng một chip đơn.

OCV là vấn đề ngày càng nghiêm trọng từ 130nm trở xuống, và tệ hơn ở các process nodes nhỏ hơn.

**Ba chế độ timing analysis:**

|Mode|Cơ chế|
|---|---|
|**Single**|Dùng một bộ delay duy nhất từ một library set cho cả setup và hold checks.|
|**Best-Case/Worst-Case (BC-WC)**|Dùng max delay cho tất cả paths khi check setup; dùng min delay cho tất cả paths khi check hold.|
|**OCV**|Dùng max delay cho launch path và min delay cho capture path (cho setup check); và ngược lại cho hold check. Mỗi path có thể được analyzed với delay khác nhau — mô phỏng thực tế nhất.|

**Lý do dùng OCV:** Trong thực tế, cùng một thời điểm, launch path và capture path trải qua các điều kiện process khác nhau (do on-chip variation). OCV mô phỏng worst-case scenario này.

### 16.6 Common Path Pessimism Removal (CPPR)

Trong OCV mode, **clock common path** (phần clock tree chung giữa launch path và capture path trước khi phân nhánh) bị analyze với hai delay khác nhau — điều này là **không thực tế** vì cùng một đoạn dây không thể có hai delay khác nhau cùng lúc.

**CPPR** loại bỏ sự pessimism này bằng cách điều chỉnh lại giá trị clock skew:

**Ví dụ:**

- Launch path delay = 1 + 1 + 1 = 3 ns (max)
- Capture path delay = 0.8 + 0.8 + 0.8 = 2.4 ns (min)

**Không có CPPR:** $$\text{Clock Skew} = 3 - 2.4 = 0.6\text{ ns}$$

**Phần common path** (đến điểm phân nhánh n1) có delay: max = 1+1 = 2 ns, min = 0.8+0.8 = 1.6 ns.

**CPPR adjustment value:** $$\Delta_{CPPR} = (1+1) - (0.8+0.8) = 0.4\text{ ns}$$

**Với CPPR:** $$\text{Clock Skew}_{adjusted} = 3 - 2.4 - 0.4 = 0.2\text{ ns}$$

CPPR cho kết quả thực tế hơn → ít pessimistic hơn → timing analysis chính xác hơn → tránh over-fix không cần thiết.

---

## 17. Các Phương Pháp Tối Ưu Timing (Pre-CTS Methods)

### 17.1 Buffering

Quá trình thêm Buffer hoặc Inverter vào net. Áp dụng cho:

- Fix setup timing violations (tăng tốc signal propagation bằng cách "regenerate" signal).
- Fix max transition violations.
- Fix max capacitance violations.

**Nguyên lý:** Buffer tách net dài thành các đoạn ngắn hơn, mỗi đoạn có capacitance nhỏ hơn → slew tốt hơn, delay thấp hơn.

### 17.2 Cell Sizing

Quá trình thay đổi drive strength của cell (lên larger hoặc smaller):

- **Upsize:** Dùng cell lớn hơn (drive strength cao hơn) → tăng tốc → fix setup timing, max transition, max capacitance.
- **Downsize:** Dùng cell nhỏ hơn → giảm power, giảm area (đánh đổi với timing).

**Nguyên lý:** Cell với drive strength lớn hơn có output resistance nhỏ hơn → charge/discharge load capacitance nhanh hơn → slew tốt hơn → delay nhỏ hơn.

### 17.3 VT Swapping (Threshold Voltage Swapping)

### 17.3.1 Standard Cell VT Types

Mỗi Standard Cell có thể được implement với các mức threshold voltage (VT) khác nhau, nhưng **cùng footprint** (cùng kích thước, cùng vị trí pin):

|VT Type|Performance|Power|
|---|---|---|
|**Low VT (LVT)**|Cao nhất (fastest)|Cao nhất (most power)|
|**Standard/Regular VT (SVT/RVT)**|Trung bình|Trung bình|
|**High VT (HVT)**|Thấp nhất (slowest)|Thấp nhất (least power)|

**Lý do:** Transistor với VT thấp → ON nhanh hơn → drive current lớn hơn → faster switching → nhưng leakage current cao hơn.

**Quan trọng:** Vì tất cả VT cells có cùng footprint, VT swapping **không ảnh hưởng đến Placement hoặc Routing** — chỉ thay đổi library file reference.

### 17.3.2 Ứng dụng VT Swapping

- **Fix setup timing violations (giai đoạn cuối):** Swap LVT cho cells trên critical path → tăng tốc.
- **Fix hold timing violations (giai đoạn cuối):** Swap HVT → tạo thêm delay.
- **Power reduction:** Swap HVT cho cells trên non-critical paths → giảm leakage power mà không ảnh hưởng timing.

### 17.4 Logic Restructuring

Thay đổi cấu trúc logic mà không thay đổi chức năng:

- Thay đổi cell types hoặc loại bỏ Buffer/Inverter dư thừa.
- **Gate decomposition:** Phân tách một cell phức tạp thành nhiều cells đơn giản hơn (ví dụ: AOI → AND + OR + INV). Dùng khi cần reduce logic depth.
- **Gate composition:** Gộp nhiều cells đơn giản thành một cell phức tạp hơn (ví dụ: AND + OR + INV → AOI). Dùng để reduce area và timing.

**Ứng dụng:** Fix setup timing violations; giảm area.

### 17.5 Cloning

Quá trình nhân bản (replicate) logic để giảm output loading:

**Vấn đề:** Một cell drive quá nhiều sinks → fanout load cao → slow transition.

**Giải pháp:** Clone cell → tạo ra 2 (hoặc nhiều) bản copy của cell, mỗi bản drive một phần sinks → giảm fanout trên mỗi driver.

**Ứng dụng:** Fix setup timing violations; fix max fanout violations.

**Khác với Buffering:** Buffering giữ nguyên số lượng driver cells, thêm Buffers. Cloning tạo thêm bản copy của chính driver cell đó.

### 17.6 Pin Swapping

Quá trình hoán đổi các input pins của cell có chức năng tương đương (equivalent pins) để cải thiện timing hoặc power:

- **Fix setup timing:** Di chuyển timing critical net sang input pin có đường đi nhanh nhất (shortest logical effort).
- **Giảm power dissipation:** Di chuyển high toggle rate net sang pin có input capacitance nhỏ hơn → giảm switching power ($P = \alpha C V^2 f$).

---

## 18. MCMM và Analysis Views

**Multi-Corner Multi-Mode (MCMM)** cho phép analyze timing dưới nhiều điều kiện vận hành đồng thời:

- **Corner:** Tổ hợp process (fast/slow), voltage (high/low), temperature (hot/cold). Ví dụ: slow corner = slow process + low voltage + high temperature (worst-case cho setup).
- **Mode:** Functional mode, scan mode, v.v. — mỗi mode có SDC constraints khác nhau.
- **Analysis View:** Tổ hợp của một constraint mode và một delay corner.

**Cho Pre-CTS Setup Optimization:**

- Active setup view: **max delay corner** (worst-case) + functional constraint mode.
- Hold view cũng được set nhưng không được optimize ở Pre-CTS stage.

---

## 19. RC Extraction

**RC Extraction** là quá trình trích xuất resistance (R) và capacitance (C) của tất cả interconnects/wires trong design từ layout geometry.

**Tại sao cần RC Extraction?** Timing analysis sử dụng wire delay model dựa trên Elmore Delay hoặc SPICE-based models — cả hai đều cần R và C của wire. Độ chính xác của timing tăng dần theo stages: wire estimation → global route RC → full RC extraction.

**Ba định dạng output:**

- **DSPF (Detail Standard Parasitic Format):** Chi tiết nhất, file lớn nhất.
- **RSPF (Reduced Standard Parasitic Format):** Đã được simplify.
- **SPEF (Standard Parasitic Extraction Format):** Compact nhất, phổ biến nhất, được hỗ trợ bởi tất cả tools. Format chuẩn ngành.

**Ứng dụng của SPEF:** Timing analysis, simulation, cross-talk analysis.

**Ở giai đoạn Placement:** RC extraction dùng **pre-route mode** — ước tính RC dựa trên wire topology dự kiến, chưa có actual routing. Kết quả được dùng cho Pre-CTS timing optimization.

---

## 20. Concurrent Placement và Timing Optimization

**Traditional Flow:**

1. Chạy Placement → `place_design`
2. Sau đó chạy Pre-CTS timing optimization → `opt_design -pre_cts`

**Concurrent Approach:**

- Chạy Placement và timing optimization đồng thời → `place_opt_design`

**Tại sao Concurrent tốt hơn?**

- Placement decisions được đưa ra với timing awareness từ đầu — cell được đặt gần nhau không chỉ dựa trên net connectivity mà còn dựa trên timing criticality.
- Giảm số lần iteration giữa Placement và Optimization.
- Cải thiện PPA convergence tổng thể: timing-aware placement → ít timing violations hơn ngay từ đầu → ít cần buffer insertion → mật độ cell thấp hơn → routing dễ hơn.

---

## 21. Kết Nối Tổng Thể Giữa Các Khái Niệm

```
Technology Library (LEF, .lib, RC)
        ↓
SITE → ROW → Placement Area (Core)
        ↓
NETLIST (Cells + Nets)
        ↓
[Pre-Placement Sanity Checks]
        ↓
GLOBAL PLACEMENT
(wirelength optimization, overlap allowed)
        ↓
GLOBAL ROUTING (GCELL estimation, overflow check)
        ↓
DETAILED PLACEMENT
(legalization, align to sites/rows)
        ↓
HFNS (fix high fanout, insert buffers)
        ↓
SCAN CHAIN REORDERING (reduce scan wirelength)
        ↓
DRV FIXING (max_cap, max_tran, max_fanout)
[Methods: Buffering, Cell Sizing]
        ↓
PRE-CTS TIMING OPTIMIZATION
[Methods: Buffering, Sizing, VT-swap, Restructuring, Cloning, Pin-swap]
[OCV + CPPR for accuracy]
        ↓
RC EXTRACTION (SPEF) → Timing Report (WNS/TNS/FEP)
        ↓
[Iterate if needed]
        ↓
→ CTS Stage
```

**Chuỗi nhân quả cốt lõi:**

- Placement density cao → routing congestion cao → timing khó meet (routing detour → delay tăng).
- Cell đặt gần nhau (net-aware) → wirelength ngắn → delay nhỏ → timing tốt hơn.
- DRV không được fix → slew chậm → propagated delay lớn → timing fail.
- HFNS không được thực hiện → reset/control signal quá chậm → timing fail trên toàn design.
- Scan chain không reorder → routing congestion xung quanh scan nets → ảnh hưởng toàn bộ routing.