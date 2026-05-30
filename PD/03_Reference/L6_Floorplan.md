# Floorplanning (Part II) — Phân Tích Lý Thuyết Toàn Diện

## 1. Vị Trí Trong Floorplanning Flow

Lecture 6 tiếp nối từ Lecture 5, tập trung vào các bước còn lại sau Place Macros:

```
Define Boundary → Placed IOs → Place Macros → [Define SC Areas] → [Build Power Grid] → [Check Quality] → Go to Placement
```

Ba bước được đánh dấu là trọng tâm của Lecture 6. Flow có tính **iterative**: vòng lặp FP Iterations xảy ra từ bước "Define SC Areas" đến "Check Quality" — designer lặp lại cho đến khi quality đạt yêu cầu, sau đó mới chuyển sang Placement.

---

## 2. Define Standard Cell Placement Areas

### 2.1 Tại Sao Phải Define SC Placement Areas

Bước này có bốn mục đích cốt lõi:

**① Enables legal cell placement:** Standard Cell bắt buộc phải được đặt trên các legal site, căn chỉnh đúng với row structure do công nghệ quy định. EDA tool sẽ không đặt bất kỳ Standard Cell nào ra ngoài vùng này.

**② Supports efficient utilization of the core area:** Sử dụng hiệu quả Core Area mà không vi phạm foundry rule.

**③ Facilitates DRC-lean routing:** Reserving đủ không gian cho signal, power, và clock routing — giảm thiểu DRC violation về sau.

**④ Facilitates Timing Closure:** Vùng SC placement được định nghĩa đúng là nền tảng để đạt Timing Closure trong các bước tiếp theo.

---

### 2.2 Standard Cell Placement Area và Placement Rows

**Standard Cell Placement Area** là vùng được xác định rõ ràng trong Core, trong đó EDA tool chỉ được phép đặt Standard Cell. Tool sẽ **không** place Standard Cell ra ngoài ranh giới đỏ (PR Boundary) của vùng này.

**Placement Rows** là các dải ngang (horizontal band) cách đều nhau, tạo thành các legal region mà Standard Cell có thể được placed vào. Đặc điểm:

- Mỗi row được cấu thành từ các **legal site**, kích thước và alignment của site phải khớp với thư viện Standard Cell.
- Row được căn chỉnh theo **technology's legal site grid**.
- Cấu trúc row được xác định bởi **track height** — ví dụ 7-track, 9-track tương ứng với số lượng routing track trong một cell height.
- Cell có **chiều cao đồng nhất** (Uniform Height) và **chiều rộng là bội số của Site Width**.

**Row Orientation** xen kẽ giữa R0 và MY:

|Orientation|Ý nghĩa|
|---|---|
|**R0**|Upright — không rotation, không mirror|
|**MY**|Flip top-to-bottom (lật theo trục Y)|

Việc xen kẽ R0 và MY cho phép **cell flipping** — các cell ở hai row liền kề chia sẻ PG Rail và N-Well ở biên giữa hai row (abutted), giúp tối ưu hóa mật độ và giảm số lượng via cần thiết cho power delivery.

**Cấu trúc tại ranh giới giữa hai row liền kề:**

```
Row (MY):   VDD Rail ← trên cùng
            NWELL (shared)
Row (R0):   VDD Rail ← trên cùng  
            ...
            VSS Rail ← dưới cùng
```

PG Rail và N-Well được **abutted** (áp sát nhau) tại biên giữa hai row, tạo continuity cho power distribution.

---

### 2.3 Wiring Tracks

**Wiring Tracks** là tập hợp các đường grid routing cách đều nhau, định hướng chính xác nơi wire có thể được đặt trong quá trình signal routing.

Đặc điểm cốt lõi:

- Mỗi track được **căn chỉnh theo preferred routing direction** của từng metal layer. Ví dụ: M2 có preferred direction là horizontal → M2 track chạy theo chiều ngang; M3 có preferred direction là vertical → M3 track chạy theo chiều dọc.
- **Track pitch** được xác định bởi hai yếu tố: design rule (metal width + spacing) và manufacturing requirement.
- Wiring track tồn tại trên toàn bộ Standard Cell Placement Area — là hạ tầng routing foundation cho toàn bộ signal Net.

**Mối quan hệ Track Pitch — Layer:**

$$\text{Track Pitch} = \text{Metal Width} + \text{Metal-to-Metal Spacing}$$

Track pitch nhỏ hơn → nhiều track hơn trong cùng diện tích → routing resource dồi dào hơn, nhưng DRC margin mỏng hơn.

---

### 2.4 Placement Blockages

Có bốn loại Placement Blockage, mỗi loại có mức độ restrictive khác nhau:

|Loại|Hành vi|
|---|---|
|**Hard Blockage**|Chặn hoàn toàn mọi placement trong vùng này|
|**Partial Blockage**|Cho phép placement đến mức utilization tối đa nhất định (ví dụ: 50%)|
|**Soft Blockage**|Chỉ cho phép một số loại Cell nhất định như Buffer, Inverter, Clock Gater|
|**Macro-only Blockage**|Chặn placement của Macro; Standard Cell vẫn được phép|

**Halo** là một dạng blockage đặc biệt xung quanh Macro:

- Kích thước cố định, gắn trực tiếp với một Macro cụ thể.
- **Halo di chuyển cùng Macro** — khi Macro bị di chuyển, Halo tự động dịch chuyển theo, đảm bảo khoảng cách tối thiểu giữa Macro và Standard Cell luôn được duy trì.
- Mục đích: dành routing channel và tránh congestion ngay cạnh Macro.

---

### 2.5 Routing Blockages

Routing Blockage là vùng do designer định nghĩa, ngăn hoặc giới hạn signal routing trên các metal layer được chỉ định trong vùng đó của Floorplan.

**Mục đích:**

- Hướng dẫn router tránh các sensitive region (ví dụ: vùng gần analog block, vùng gần I/O Pad).
- Preserve routing resource cho các mục đích cụ thể.
- Enforce design constraint (ví dụ: không cho route signal qua vùng power strap).

**Phân biệt cơ chế quản lý congestion:**

|Phương pháp|Cơ chế|
|---|---|
|Placement Blockage|Gián tiếp: giảm cell → giảm wire demand|
|Routing Blockage|Trực tiếp: cấm route trên layer cụ thể|

---

## 3. Power Distribution Network (PDN)

### 3.1 Định Nghĩa và Mục Tiêu

Power Planning là quá trình phân tích power consumption của chip và phân phối power từ IO area đến tất cả Cell trong Core Area.

**Bốn mục tiêu của Power Planning:**

**① Providing power to all cells:** Mọi Cell đều cần power để hoạt động — đây là yêu cầu tối thiểu, không thể thương lượng.

**② Minimizing IR Drop:** Đáp ứng constraint về maximum IR drop, qua đó hỗ trợ Timing Closure (vì IR drop ảnh hưởng trực tiếp đến cell delay).

**③ Minimizing self-heating:** Không vi phạm EM (Electromigration) rule — tránh current density quá cao gây hỏng metal wire theo thời gian.

**④ Avoiding over-designing:** Tránh thiết kế PDN quá mức cần thiết. Over-designed PDN chiếm routing resource (do power stripe rộng/dày đặc), tăng die cost, và hầu như không cải thiện thêm power integrity sau một ngưỡng nhất định.

---

### 3.2 Cấu Trúc Phân Cấp của PDN

PDN có cấu trúc **hierarchical** từ I/O Pad xuống đến từng Standard Cell:

```
PG Pads (IO Ring)
    ↓
Core PG Rings (bao quanh toàn bộ Core)
    ↓
PG Stripes/Straps (lưới ngang/dọc bên trong Core)
    ↓
Macro (Block) PG Rings (bao quanh từng Macro)
    ↓
Standard Cell PG Rails (M1, trong từng SC row)
    ↓
Standard Cell VDD/VSS Pins
```

**Core PG Rings:** Metal ring rộng bao quanh Core. Được tạo từ high metal layer (ví dụ: M8, M9) để giảm resistance.

**Macro PG Rings (Block Rings):** Metal ring bao quanh từng Macro riêng lẻ — đảm bảo Macro được kết nối trực tiếp với power grid, không phụ thuộc vào Standard Cell row.

**PG Stripes/Straps:** Các metal trace dọc và ngang bên trong Core, kết nối Core Ring với Standard Cell PG Rail. Tạo thành dạng mesh hoặc grid.

**Standard Cell PG Rails:** Các thanh power chạy ngang trên M1, nằm trong từng Standard Cell row. Đây là điểm kết nối cuối cùng đến pin VDD/VSS của từng cell.

---

### 3.3 Multi-Layer Distribution

PDN sử dụng nhiều metal layer với phân công vai trò rõ ràng:

|Layer|Vai trò|
|---|---|
|M7, M8, M9 (high layer)|Global power distribution — resistance thấp, ít ảnh hưởng đến signal routing|
|M4, M5, M6 (mid layer)|Intermediate distribution — PG stripes|
|M1, M2, M3 (low layer)|Local distribution đến Standard Cell và Macro|

**Nguyên lý:** Layer càng cao → resistance sheet càng thấp → IR drop càng nhỏ. Tuy nhiên, dùng nhiều high layer cho power → ít routing resource còn lại cho signal Net.

---

### 3.4 Power/Ground Nets và Logical Connection

**Power/Ground Net** (VDD/VSS) **không tồn tại** trong incoming synthesized Netlist. Chúng được tạo ra trong quá trình Floorplanning/PDN.

Sau khi tạo Net VDD/VSS, designer phải thực hiện **logical connection** (kết nối logic) các VDD/VSS Pin của từng Cell vào đúng Net:

Có ba loại đối tượng cần được kết nối:

- **Standard Cell pins** — tất cả cell trong Core
- **Macro cell pins** — RAM, ROM và các Macro khác
- **Tie Cells** — kết nối logic constant-0 hoặc constant-1 vào power rail

---

### 3.5 Tie Cells

Khi một input Pin của Standard Cell cần được kết nối với constant-0 (`1'b0`) hoặc constant-1 (`1'b1`), **không được phép** nối trực tiếp Pin đó vào VDD hoặc VSS.

**Lý do:** Gate oxide dưới poly gate rất mỏng và là phần nhạy cảm nhất của transistor. Voltage surge trong quá trình fabrication (antenna effect) hoặc trong quá trình hoạt động có thể phá hủy gate oxide nếu nó được nối trực tiếp với power rail.

**Giải pháp:** Dùng **Tie Cell** — là Standard Cell chuyên dụng chứa diode-connected transistor. Transistor được nối theo kiểu diode nghĩa là:

$$V_g = V_d \Rightarrow V_{gs} = V_{ds} \Rightarrow V_{ds} > V_{gs} - V_t$$

Điều kiện trên đảm bảo transistor luôn ở trạng thái saturation — cung cấp mức điện áp ổn định (VDD hoặc VSS) đến output trong khi cô lập gate oxide khỏi surge trực tiếp.

- **Tie-high Cell:** Output = VDD (logic 1)
- **Tie-low Cell:** Output = VSS (logic 0)

Tie Cell là Standard Cell bình thường nhưng **không xuất hiện trong Netlist** — chúng được insert trong quá trình Floorplanning/PDN.

---

## 4. IR Drop — Lý Thuyết và Phân Tích

### 4.1 Mô Hình Điện Trở Của Power Grid

PDN có thể được model hóa như một mạng điện trở phân tán. Khi current chạy qua resistance của metal wire, sẽ xuất hiện voltage drop theo định luật Ohm:

$$V_{\text{drop}} = I \times R_{\text{wire}}$$

Điện áp thực tế tại cell = VDD (từ Pad) − Voltage drop tích lũy trên đường từ Pad đến cell. Cell hoạt động ở điện áp thấp hơn danh định → switching chậm hơn → delay tăng.

---

### 4.2 Static IR Drop

Static IR Drop được tính dựa trên **average current** — dòng điện trung bình tiêu thụ bởi toàn bộ design trong steady-state operation.

**Average Power** là tổng của ba thành phần:

$$P_{\text{avg}} = P_{\text{leakage}} + P_{\text{internal}} + P_{\text{switching}}$$

Trong đó:

$$P_{\text{leakage}} = \sum \text{cell leakage} = \text{Gate Leakage} + \text{Junction Leakage} + \text{Subthreshold Leakage}$$

$$P_{\text{internal}} = \text{Internal Energy (từ .lib)} \times f \times TR$$

$$P_{\text{switching}} = \frac{1}{2} \times C \times V^2 \times f \times TR$$

Trong đó: $f$ = clock frequency, $TR$ = Toggle Rate.

**Công thức Static IR Drop:**

$$\text{Static IR Drop} = I_{\text{avg}} \times R_{\text{wire}} = \frac{P_{\text{avg}}}{V_{DD}} \times R_{\text{wire}}$$

Static IR Drop phản ánh **worst-case average** — tương ứng với scenario toàn bộ chip đang hoạt động ở average load.

---

### 4.3 Dynamic IR Drop

Dynamic IR Drop xảy ra khi transistor đang **switching**. Khi nhiều cell chuyển trạng thái đồng thời ở tần số cao, dòng điện rút vào PDN tăng đột biến → voltage drop tăng đột ngột và dao động.

Biểu hiện:

- **Voltage droop (Power Droop):** VDD bị kéo xuống thấp hơn danh định tại thời điểm switching.
- **Ground Bounce:** VSS bị đẩy lên cao hơn 0V do current spike.

Dynamic IR Drop thường **lớn hơn** Static IR Drop (worst-case), do tính chất peak current khi nhiều cell toggle cùng lúc.

**Lưu ý về switching noise:** Nguồn gốc của supply noise bao gồm:

- **Rush current noise:** Xuất hiện khi các sleep transistor bật lên, nạp local grid capacitance → tạo current spike lớn.
- **Switching noise:** Xuất hiện do switching current của logic device trong hoạt động bình thường.

---

### 4.4 IR Drop Impacts on Timing

IR Drop ảnh hưởng đến Timing theo hai cơ chế, tùy thuộc vào vị trí xảy ra IR Drop:

**Trường hợp 1 — IR Drop trên signal path:**

- Cell trong data path hoạt động ở điện áp thấp hơn → chậm hơn.
- Tín hiệu data đến capturing Flip-flop trễ hơn dự kiến.
- **Hệ quả:** Setup time violation.

```
CLK  ─────┐          ┌─────
DATA ──────┼──[IR]──→ ↓ (delayed)
                         Setup window bị xâm phạm
```

**Trường hợp 2 — IR Drop trên clock buffer:**

- Clock buffer hoạt động chậm hơn → clock edge đến capturing Flip-flop bị trễ.
- Data đến đúng thời điểm nhưng clock edge trễ → data đến **trước** clock edge quá nhiều.
- **Hệ quả:** Hold time violation cho tất cả signal được clock bởi nhánh clock đó.

```
CLK  ──[Buffer+IR]──→ Clock edge trễ
DATA ────────────────→ Data đến sớm → Hold violation
```

---

### 4.5 Phương Pháp Giảm Static IR Drop

Khi IR Drop map cho thấy vùng đỏ (voltage drop > 10% nominal supply voltage), có thể áp dụng:

- Tăng width của power stripe.
- Thêm power stripe trên higher metal layer.
- Thêm PG Stripe bổ sung tại vùng high-current density.

---

### 4.6 Power Grid Modeling

PDN được model hóa như mạng R (resistor network) phân tán:

- Mỗi đoạn metal wire được biểu diễn bằng một điện trở: $R = \rho \times L / (W \times t)$
- Current source tại mỗi cell tap biểu thị dòng tiêu thụ của cell.
- VDD Pad là voltage source.
- Toàn bộ mạng được giải qua nodal analysis để tính voltage tại mọi điểm.

Resistance của interconnect từ M1 đến M5 tăng dần theo layer thấp hơn (sheet resistance cao hơn), nên global power stripes trên M7/M8/M9 có R thấp → IR drop nhỏ hơn trên path dài.

---

## 5. Electromigration (EM)

### 5.1 Định Nghĩa

Electromigration là hiện tượng dịch chuyển dần dần các nguyên tử kim loại trong dây dẫn bán dẫn. Xảy ra khi current density đủ cao để gây ra drift của metal ion theo hướng electron flow.

Electromigration là **wear-out mechanism** — không gây hỏng ngay lập tức mà xảy ra theo thời gian dài (có thể nhiều năm).

### 5.2 Nguyên Nhân

**Nguyên nhân cơ học:** EM là mechanical failure trong wire, do điều kiện nhiệt thay đổi thường xuyên:

- Khi xung điện chạy qua wire, năng lượng được tiêu tán dưới dạng nhiệt → wire nóng lên trên oxide temperature.
- Sự chênh lệch thermal expansion coefficient giữa oxide và metal wire tạo ra mechanical stress.
- Stress tích lũy → wire cuối cùng bị hỏng → chip failure trong điều kiện vận hành thực tế.

**Hai nguyên nhân chính gây EM failure:**

- **High DC current density:** Electron flow kéo metal ion theo hướng drift.
- **Joule heating từ alternating current cao:** Nhiệt sinh ra do $P = I^2 R$ làm tăng tốc quá trình dịch chuyển nguyên tử.

### 5.3 Hiệu Ứng Của EM

|Hiệu ứng|Cơ chế|Hậu quả|
|---|---|---|
|**Void (lỗ hổng)**|Nguyên tử bị kéo đi khỏi một vùng → tạo khoảng trống|Interconnect failure (open circuit) — giảm connectivity dần dần|
|**Hillock (gò kim loại)**|Nguyên tử tích tụ tại một vùng khác → tạo khối nhô|Short circuit hazard với wire lân cận|

Void dẫn đến **open circuit** (nguy cơ failure chức năng), còn Hillock dẫn đến **short circuit** (nguy cơ ESD hoặc functional fault).

---

## 6. Components of Power in Standard Cells

### 6.1 Dynamic Power

Dynamic Power gồm hai thành phần:

**Switching Power** (dominant component):

$$P_{\text{switching}} = \alpha \cdot C \cdot V^2 \cdot f$$

Trong đó:

- $\alpha$: Switching activity factor ($0 < \alpha \leq 1$) — tỉ lệ số transition thực tế so với tối đa.
- $C$: Load capacitance (interconnect + gate capacitance của fanout).
- $V$: Supply voltage.
- $f$: Clock frequency.

Đây là thành phần **dominant** — chiếm phần lớn tổng dynamic power, do charging/discharging load capacitance.

**Internal Power:**

$$P_{\text{short\_circuit}} \approx I_{sc} \cdot V \cdot t_{sc} \cdot f$$

Trong đó:

- $I_{sc}$: Short-circuit current trong thời gian transition.
- $t_{sc}$: Khoảng thời gian cả P và N transistor cùng dẫn điện.

Internal Power bao gồm hai sub-component:

- **Short-circuit power:** Xảy ra khi cả PMOS và NMOS cùng ON trong một khoảnh khắc ngắn của signal transition, tạo direct path từ VDD xuống GND.
- **Internal node switching:** Nạp/xả các node nội bộ của cell.

### 6.2 Static Power (Leakage)

$$P_{\text{leakage}} = V \cdot I_{\text{leak}}$$

Leakage không phụ thuộc vào switching activity, xảy ra ngay cả khi chip idle.

Leakage phụ thuộc vào:

- Cell type: HVT (High-Vth) < SVT (Standard-Vth) < LVT (Low-Vth) — cell Vth thấp hơn thì leakage cao hơn nhưng performance nhanh hơn.
- Temperature: nhiệt độ cao → tất cả leakage tăng.
- Process: biến đổi theo process corner.

**Tổng công thức:**

$$P_{\text{total}} = P_{\text{static}} + P_{\text{dynamic}} = P_{\text{leakage}} + P_{\text{internal}} + P_{\text{switching}}$$

---

### 6.3 Leakage Power Components (Chi Tiết)

Ba nguồn leakage chính trong transistor:

**① Gate Leakage Current:**

- Nguyên nhân: Electron tunneling qua gate oxide rất mỏng (đặc biệt quan trọng ở advanced node).
- Xảy ra do electric field mạnh trên gate oxide khi transistor ON hoặc OFF.
- Dòng rò chảy qua oxide kể cả khi transistor ở trạng thái lý tưởng OFF.

**② Subthreshold Leakage Current:**

- Xảy ra khi $V_{GS} < V_{th}$ — transistor về lý thuyết là OFF nhưng vẫn có weak conduction từ drain sang source.
- Là **nguồn leakage lớn nhất** ở công nghệ scaled (sub-100nm), vì Vth bị giảm để duy trì performance.
- Phụ thuộc mạnh vào nhiệt độ và process corner.

**③ Reverse-Biased Junction Leakage:**

- Xảy ra tại các PN junction nguồn/máng (source/drain) và substrate.
- Junction được reverse-biased → cho qua dòng nhỏ.
- Xuất hiện khi: NMOS source/drain ở VDD, hoặc PMOS source/drain ở VSS.

---

## 7. Latch-Up

### 7.1 Cơ Chế Latch-Up

Trong cấu trúc CMOS bulk, tồn tại một parasitic SCR (Silicon Controlled Rectifier) — hay còn gọi là cấu trúc PNPN — hình thành từ sự kết hợp của P+ source/drain, N-well, P-substrate, và N+ source/drain.

Cấu trúc này bao gồm hai BJT ký sinh:

- **Lateral NPN transistor:** N+ (source NMOS) → P-substrate → N-well
- **Vertical PNP transistor:** P+ (source PMOS) → N-well → P-substrate

**Trigger Example trên Inverter:** Khi output undershoot xuống dưới GND ($V_{\text{OUT}} < \text{GND} - 0.7V$):

1. Current inject vào p-substrate.
2. Voltage tăng trên substrate resistance $R_s$.
3. Parasitic NPN turn ON.
4. NPN current gây voltage drop trong N-well.
5. Parasitic PNP turn ON.
6. Cấu trúc PNPN latch — tạo path dẫn điện liên tục từ VDD xuống GND.
7. **Large VDD-to-GND current flows** — có thể phá hủy chip.

**Đây là classic SCR (latch-up) structure.** Khi đã latch, chỉ có thể reset bằng cách ngắt điện.

### 7.2 Phòng Ngừa Latch-Up

Lý do phải insert **Tap Cell** và tuân thủ khoảng cách foundry rule:

- Tap Cell cung cấp well và substrate contact gần với cell, giảm điện trở $R_s$ và $R_w$.
- $R_s$ và $R_w$ nhỏ hơn → voltage drop nhỏ hơn → khó trigger parasitic BJT hơn → giảm nguy cơ latch-up.

---

## 8. Tap Cells và Boundary Cells

### 8.1 Tap Cells

Có hai loại Standard Cell về phương diện well tap:

|Loại|Đặc điểm|
|---|---|
|**Traditional Standard Cell**|Có tích hợp sẵn well tap bên trong cell body — N-well và P-substrate được kết nối trực tiếp.|
|**Tapless Standard Cell**|Không có well tap tích hợp — cần Tap Cell riêng biệt.|

**Tap Cell** là Standard Cell chuyên dụng chỉ chứa well tap, không có logic. Được insert theo foundry rule với **pattern stagger (so le)** và khoảng cách quy định (ví dụ: 60 µm) — để đảm bảo mọi điểm trên chip đều trong phạm vi tapping distance.

Mục đích Tap Cell: ngăn latch-up và đảm bảo well/substrate connection đúng foundry requirement.

### 8.2 Boundary Cells

Boundary Cell được insert tại ranh giới giữa các vùng khác nhau. Mục đích: đảm bảo Cell nằm ở ranh giới giữ nguyên electrical behavior đã được characterized trong .lib.

Có tầm quan trọng đặc biệt tại **advanced node** — vì ở advanced node, các hiệu ứng well proximity effect, OD (diffusion) density requirement, và N-well continuity rule trở nên nghiêm ngặt hơn nhiều. Boundary Cell cung cấp correct N-Well và diffusion environment cho Cell biên giới.

---

## 9. Floorplanning Quality Checks

### 9.1 Tổng Quan

Trước khi chuyển sang Placement, Floorplan hoàn chỉnh phải pass checks trong **sáu category**:

1. Power Grid Integrity
2. I/O Pads/Pins and Boundary Cells
3. Macro Placement
4. Placement Rows and Core Utilization
5. Blockage Planning
6. General Sanity Checks

---

### 9.2 Category 1: Power Grid Integrity Checks

**Structure Checks:**

_Power Connectivity:_

- Tất cả Standard Cell row phải kết nối với M1 PG Rail.
- Macro và Pad phải kết nối với PG mesh.

_PG Design Rules:_

- **Tap Cell Insertion:** Nếu dùng Tapless Standard Cell, Tap Cell phải được insert theo latch-up rule trước khi chuyển sang Placement.
- **PG Shape DRC:** Kiểm tra floating PG metal/via shape, short, spacing error.

_PG Stripe and Ring Coverage:_

- Xác minh PG Stripe và Ring cover toàn bộ Core — thêm Macro Ring nếu cần.
- Kiểm tra gap trong coverage gần Macro hoặc I/O.

**Early EMIR Checks:**

- Estimate PG grid quality trước khi routing diễn ra.
- Run IR Drop và EM analysis.
- Mục đích: phát hiện sớm Power Grid weakness để fix trong Floorplan iteration, tránh phải rework sau khi đã Placement và Routing.

**Early Rail Analysis Flow:**

```
Power Planning Started
→ Early Plan of Power Grid Lines and Width
→ Power Grid Modeling
→ Power Grid Analysis
→ [IR drop/EM trong allowed margin?]
   Yes → Power Planning Ends
   No  → Fix it → quay lại Power Grid Modeling
```

---

### 9.3 Category 2: I/O Pads/Pins và Boundary Cells

- Không có DRC error trong I/O Pad/Pin placement.
- I/O Pad được kiểm tra theo packaging rule và ESD requirement.
- Boundary Cell (End Cap, Tap Cell) đã được insert theo yêu cầu.
- Seal Ring nguyên vẹn và đúng specification.
- Pin được đặt dọc theo và nằm trong cạnh của PR Boundary.

---

### 9.4 Category 3: Macro Placement

- Data flow được xem xét cho timing/power-critical Macro — sử dụng flight line để phân tích connectivity; không có routing detour đáng kể.
- Macro được đặt gần I/O Pad hoặc interface logic liên quan.
- Sensitive Macro (analog, RF) được isolated khỏi noisy digital hoặc I/O circuit.
- Synchronous Macro được đặt gần related clock generation circuit.
- Không có highly congested area như chimney, high-pin Macro clustering.
- Không có clustering của power-hungry Macro.

---

### 9.5 Category 4: Placement Row và Core Utilization

- Row được định nghĩa với correct orientation và nằm trong legal Standard Cell area (không overlap Macro).
- Wiring track được căn chỉnh theo preferred routing direction.
- Track pitch được định nghĩa đúng.
- Core utilization ở mức reasonable (thông thường 65–80%).
- Không có large unused space trên die.
- Placement area phải **as contiguous as possible** — tránh bị chia cắt thành nhiều mảnh nhỏ.

---

### 9.6 Category 5: Blockage Planning

- Sử dụng Placement Blockage và/hoặc Routing Blockage để giải quyết congestion hoặc timing issue.
- Kiểm tra không có thừa/thiếu blockage không cần thiết.

---

### 9.7 Category 6: General Sanity Checks

Kiểm tra tổng quát toàn bộ Floorplan:

- Routing track đúng cấu hình.
- FINFET Grid (nếu applicable) on Manufacture Grid.
- Core/die box on grid.
- Snap rule.
- Row on grid.
- AreaIO row.
- Row out of die.
- Routing blockage.
- Placement blockage.
- Components (không snap đến row-site).
- IO Pad out of die.
- Constraints (guide/region/fence).
- Pre-routes on track.

---

## 10. Các Metric Đánh Giá Floorplan Quality

### 10.1 Core Utilization

$$\text{Utilization Ratio} = \frac{\text{Total Area of Cells}}{\text{Total Capacity Area}}$$

Trong đó Total Capacity Area **loại trừ**: hard macros, macro keepouts, soft macros, io cells, hard blockages.

Utilization hợp lý ở mức khoảng 0.60–0.80 (60–80%). Utilization quá cao (>85%) dẫn đến routing congestion, timing closure khó khăn. Utilization quá thấp lãng phí die area và tăng cost.

### 10.2 Routing Congestion

Routing congestion được đánh giá qua Global Routing Congestion map:

$$\text{Congestion} = \frac{\text{Routing Demand}}{\text{Routing Supply}} \implies \text{Overflow} = \max(0, \text{Demand} - \text{Supply})$$

Báo cáo congestion phân chia theo direction:

- **H routing (Horizontal):** Average và peak track utilization theo hướng ngang.
- **V routing (Vertical):** Average và peak track utilization theo hướng dọc.

Overflow = 0 ở mọi GRC (Global Routing Cell) là trạng thái lý tưởng. Overflow > 0 tại vùng nào đó báo hiệu routing infeasibility tiềm năng.

---

## 11. Tổng Hợp — Mối Liên Kết Giữa Các Khái Niệm

```
FLOORPLANNING (Part II)
│
├── DEFINE SC AREAS
│   ├── Placement Rows: legal regions cho SC, R0/MY orientation, site grid
│   ├── Wiring Tracks: routing grid, preferred direction per layer, pitch = width + spacing
│   ├── Placement Blockage: Hard / Partial / Soft / Macro-only / Halo
│   └── Routing Blockage: ngăn route trên specified layer → complements Placement Blockage
│
├── BUILD POWER GRID (PDN)
│   ├── Mục tiêu: Cấp power ↔ Minimize IR Drop ↔ Minimize EM ↔ Avoid over-design
│   ├── Cấu trúc phân cấp: PG Pad → Core Ring → Stripes → Block Ring → SC Rail → Cell Pin
│   ├── Multi-layer: High layer (global, low R) → Low layer (local distribution)
│   ├── Power Nets: VDD/VSS không có trong Netlist → phải create và connect
│   ├── Tie Cells: thay thế direct VDD/VSS connection → bảo vệ gate oxide
│   │
│   ├── IR DROP
│   │   ├── Static IR Drop = I(avg) × R = [P(avg)/VDD] × R
│   │   │   P(avg) = P(leakage) + P(internal) + P(switching)
│   │   ├── Dynamic IR Drop = voltage drop khi transistors switching đồng thời
│   │   └── Impact on Timing:
│   │       ├── IR drop trên signal path → Setup violation
│   │       └── IR drop trên clock buffer → Hold violation
│   │
│   └── ELECTROMIGRATION
│       ├── Nguyên nhân: high current density, Joule heating từ alternating current
│       ├── Effects: Void (open circuit) ↔ Hillock (short circuit)
│       └── Liên kết với PDN: stripe width phải đủ để J < J_max (EM limit)
│
├── POWER COMPONENTS (lý thuyết nền)
│   ├── Switching: α·C·V²·f (dominant)
│   ├── Short-circuit: Isc·V·tsc·f (khi cả P và N ON đồng thời)
│   ├── Internal node switching
│   └── Leakage: Gate oxide tunneling + Subthreshold + Junction reverse bias
│       → phụ thuộc Vth, nhiệt độ, process → ảnh hưởng chọn HVT/SVT/LVT
│
├── LATCH-UP
│   ├── Parasitic PNPN SCR trong bulk CMOS
│   ├── Trigger: output overshoot/undershoot → inject current → latch
│   └── Phòng ngừa: Tap Cell giảm Rsubstrate và Rwell → khó trigger
│
└── QUALITY CHECKS (gate to Placement)
    ├── 1. Power Grid Integrity: connectivity, DRC, coverage, early EMIR
    ├── 2. I/O Pads/Pins/Boundary Cells: DRC, ESD, Seal Ring, Pin location
    ├── 3. Macro Placement: data flow, congestion, isolation, clock proximity
    ├── 4. Placement Row & Core Utilization: orientation, track, utilization, contiguity
    ├── 5. Blockage Planning: coverage, necessity
    └── 6. General Sanity: grids, snap rules, pre-routes
```
