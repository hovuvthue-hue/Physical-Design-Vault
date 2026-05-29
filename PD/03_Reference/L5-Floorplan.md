# Floorplanning (Part I) — Phân Tích Lý Thuyết Toàn Diện

## 1. Tổng Quan về Floorplanning trong Physical Design Flow

### 1.1 Vị trí trong PD Flow

Floorplanning là **bước major đầu tiên** trong physical design (PD) flow, nằm giữa Design Import và Placement. Toàn bộ flow có thứ tự:

```
Design Import → Floorplanning → Placement → CTS → Routing → Design Export → Fab
```

Floorplanning có tính chất **iterative** — engineer thường phải lặp lại nhiều lần trước khi chuyển sang Placement.

### 1.2 Định nghĩa

Floorplanning xác định **physical layout** của chip hoặc block, bao gồm:

- Vị trí của các functional block lớn
- Vị trí của các Macro
- Vị trí của I/O Pad và I/O Pin

---

## 2. Mục Tiêu của Floorplanning

Floorplanning hướng đến 6 mục tiêu chính, có mối quan hệ chặt chẽ với nhau:

**① Define Optimal Physical Boundaries** Xác định ranh giới die (hoặc block) để chứa toàn bộ logic, Macro và hạ tầng một cách hiệu quả.

**② Meet Best PPA Results** Tối ưu Placement để cân bằng **Performance – Power – Area (PPA)** trên toàn bộ design.

**③ Enable Clean and Legal I/O Placement** Đặt I/O Pad (hoặc Pin) và kết nối power/ground để hỗ trợ top-level integration.

**④ Place Macros and Plan Standard Cell Regions** Sắp xếp Macro và xác định placement row để đạt Timing, Utilization và Routing hiệu quả nhất.

**⑤ Build a Robust Power Delivery Network (PDN)** Thiết kế power ring, strap hoặc mesh để đáp ứng yêu cầu IR drop và electromigration (EM).

**⑥ Create Custom Pre-Routes for Critical Nets (nếu cần)** Route thủ công (bằng tay hoặc script) các Net quan trọng như clock, reset, analog — những Net yêu cầu kiểm soát path cụ thể, shielding, symmetry hoặc topology mà automated routing không đáp ứng được.

---

## 3. Các Bước Chính trong Floorplanning

```
Define Boundary → Placed IOs → Place Macros → Define SC Areas → Build Power Grid → Check Quality → Go to Placement
```

Mỗi bước có thể được lặp lại (FP Iterations) cho đến khi đạt chất lượng mong muốn.

---

## 4. Define Top-Level PnR Boundary

### 4.1 Các Khái Niệm Cốt Lõi

|Khái niệm|Định nghĩa|
|---|---|
|**Die Area**|Tổng diện tích silicon được cấp phát cho chip|
|**Core Area**|Vùng bên trong Die nơi Standard Cell và logic được Placed|
|**I/O Ring Area**|Vùng bao quanh Core, dành để đặt I/O Cell|
|**Core-to-Ring Spacing**|Khoảng cách giữa Core boundary và I/O Ring|
|**Aspect Ratio**|Tỉ lệ chiều rộng/chiều cao của die|

**Mối quan hệ không gian:**

```
[IO Ring Area] bao ngoài [Core-to-IO Spacing] bao ngoài [Core Area]
Core Area = Standard Cell Area + Macro Area
```

### 4.2 Công Thức Tính Die Area (Chip Size)

$$\text{Die Area} = \text{IO Ring Area} + \text{Core-to-IO Spacing Area} + \text{Core Area}$$

Trong đó:

- **IO Ring Area** được xác định bởi: chiều cao IO Cell × số lượng Pad
- **Core-to-IO Spacing** được xác định bởi: Design Rule (phòng ngừa latch-up)
- **Core Area** là phần **duy nhất** mà designer có thể kiểm soát kích thước

### 4.3 Aspect Ratio

$$\text{Aspect Ratio} = \frac{\text{Width}}{\text{Height}}$$

|Giá trị|Ý nghĩa|
|---|---|
|= 1.0|Hình vuông|
|> 1.0|Rộng (wide)|
|< 1.0|Cao (tall)|

**Lý do điều chỉnh Aspect Ratio:**

- **Fit Package Constraints:** Phù hợp bonding rule, bump pitch, package shape, số lượng I/O yêu cầu.
- **Improve Routing Efficiency:** Chọn horizontal vs. vertical routing bias để giảm congestion; ưu tiên hướng bus hoặc tận dụng layer resource.
- **Improve Timing:** Cân bằng phân bố critical path theo hướng dominant data path; giảm wire length trên timing-sensitive net.

> **Usage Tip từ tài liệu:** Nếu có nhiều horizontal routing layer hơn vertical, và block cần nhiều routing resource, block nên được thiết kế ngắn và rộng (short and wide).

### 4.4 Block PnR Boundary (so với Full-Chip)

Block PnR Boundary có 3 đặc điểm khác biệt:

**① Inherited from full-chip floorplan:** Block designer không tự định nghĩa — họ nhận boundary từ top-level integration.

**② No I/O Pad:** Vị trí và hình dạng I/O Pin được định nghĩa sẵn từ top level, phải được tuân thủ chính xác.

**③ Typically rectilinear:** Được định nghĩa là vùng hình chữ nhật hoặc rectilinear. Cadence Innovus (Fusion Compiler) hỗ trợ nhiều dạng block boundary khác nhau.

---

## 5. Ước Tính Core Area

### 5.1 Công Thức Ước Tính

Core Area bao gồm nhiều thành phần:
$$\text{Core Area} = \underbrace{A_{\text{macro}} + A_{\text{SC}}}_{\text{Từ Synthesis}} + \underbrace{A_{\text{SC\_additional}}}_{\text{CTS + Timing Opt (10\%–20\%)}} + \underbrace{A_{\text{blockage}} + A_{\text{unusable}} + A_{\text{empty}}}_{\text{Macro Padding, DRC, Utilization}}$$

**Giải thích từng thành phần:**

- $A_{\text{macro}}$: Diện tích Macro lấy từ Synthesis report
- $A_{\text{SC}}$: Diện tích Standard Cell lấy từ Synthesis report
- $A_{\text{SC\_additional}}$ (10%–20%): Vùng bổ sung cho CTS, Timing optimization, Spare cell, Routing congestion relief
- $A_{\text{blockage}}$: Diện tích Placement halo, Blockage cho Macro padding
- $A_{\text{unusable}}$: Diện tích không dùng được do Design Rule hoặc irregular Floorplan
- $A_{\text{empty}}$: Vùng trống do Target Utilization thấp

### 5.2 Standard Cell Utilization

$$\text{Standard Cell Utilization} = \frac{\text{Total Standard Cell Area}}{\text{Placeable Standard Cell Area}}$$

Trong đó:

$$\text{Placeable SC Area} = A_{\text{SC}} + A_{\text{SC\_additional}} + A_{\text{empty\_SC}}$$

> Đây là metric **có ý nghĩa nhất** (Most meaningful). Utilization cao dẫn đến Routability và Signal Integrity (SI) bị ảnh hưởng xấu.

### 5.3 Quy Trình Ước Tính Core Area

**Bước 1 — Estimate Standard Cell Area:** Dùng Synthesis report để lấy tổng cell area của Standard Cell, sau đó cộng thêm 10–20% overhead cho: CTS, Timing optimization, Spare Cell, Routing congestion relief.

**Bước 2 — Choose Target Utilization:** Thông thường chọn trong khoảng 65–80%.

**Bước 3 — Compute Core Area:**

$$\text{Core Area} = \frac{\text{Estimated Cell Area}}{\text{Target Utilization}}$$

### 5.4 Die Size Estimation (Công Thức Tổng Quát Hơn)

$$\text{Die Area (mm}^2) = \frac{\left[\dfrac{G \cdot (1 + T + E)}{D}\right] + I + M}{\text{TU}}$$

Trong đó:

- $G$ = Gate count (không tính Macro)
- $D$ = Gate density (gates/mm²) — thông số công nghệ
- $H, V$ = Số horizontal/vertical routing layer — thông số công nghệ
- $I$ = I/O area (mm²)
- $M$ = Macro area (mm²)
- $T$ = Overhead cho CTS, timing closure (%)
- $E$ = Overhead cho ECO (%)
- $\text{TU}$ = Target Utilization (%)

### 5.5 Die Size Estimate Limitations

Kích thước die bị giới hạn bởi các yếu tố khác nhau:

|Loại|Nguyên nhân|
|---|---|
|**Core-limited**|Core Area lớn hơn IO Ring, die size do Core quyết định|
|**Pad-limited**|IO Ring Area lớn hơn Core, die size do số Pad quyết định|
|**Block-limited**|Quá nhiều Macro block, khó đặt chúng sát nhau|
|**Package-limited**|Package được chọn giới hạn kích thước die tối đa|

---

## 6. Routing Congestion và Aspect Ratio

### 6.1 Mối Quan Hệ Aspect Ratio — Routing Resource

|Đặc tính|Wide (Aspect Ratio > 1.0)|Tall (Aspect Ratio < 1.0)|
|---|---|---|
|Số Horizontal Routing Track|Nhiều hơn|Ít hơn|
|Số Vertical Routing Track|Ít hơn|Nhiều hơn|
|Horizontal Wire Length|Ngắn hơn|Dài hơn|
|Vertical Wire Length|Dài hơn|Ngắn hơn|

> **Nguyên tắc:** Nhiều track hơn = ít congestion hơn; dây ngắn hơn = delay ít hơn.

**Lưu ý định nghĩa:** "Horizontal routing track" = track chạy trái–phải, trên đó các wire dọc (như M2) được route.

---

## 7. Place I/O Pads và Pins

### 7.1 Phân Biệt Full-Chip I/O Pad và Block-Level I/O Pin

|Tiêu chí|Full-Chip I/O Pad|Block-Level I/O Pin|
|---|---|---|
|Kết nối đến|Package / PCB|Các block khác trong chip|
|Vị trí|Do designer định nghĩa|Được inherit từ top-level|
|Quy tắc đặc biệt|ESD, packaging rule|Pin spacing, signal grouping|
|Tác động|Package interface, power integrity|Top-level routing, congestion, timing|

### 7.2 Các Loại I/O Cell

**I/O Library gồm hai nhóm:**

**Standard I/O Cells:**

- **Digital I/O Buffer:**
    - Output: Cung cấp drive strength cao, level-shifting up
    - Input: Level-shifting down + ESD protection
    - Đặc điểm: Output buffer cần drive pF (không phải fF), yêu cầu inverter chain tăng fanout, short circuit current không được chấp nhận
- **Analog I/O Cell:** Truyền tín hiệu analog vào/ra chip; về cơ bản là "dây dẫn" nhưng phải có ESD protection
- **Power Supply Cell:** Cung cấp power cho I/O Ring (power distribution + ESD protection); core và I/O thường dùng nguồn riêng biệt vì I/O sink nhiều dòng và chạy ở điện áp cao hơn (ví dụ 2.5V so với 1.2V)
- **Corner Cell và Filler Cell:** Corner Cell chứa các wire 45-độ để giảm current crowding, cải thiện EM performance, nâng Signal Integrity, giảm metal density; Filler Cell lấp đầy khoảng trống giữa I/O Cell

**Special I/O Cells:**

- SSTL (Stub Series Terminated Logic)
- HSTL (High Speed Transceiver Logic)
- LVDS (Low Voltage Differential Signaling)

### 7.3 ESD Protection

ESD (Electrostatic Discharge) là một trong những vấn đề reliability quan trọng nhất trong IC industry.

**Cơ chế bảo vệ:**

- **Diode Clamps:** Kích hoạt khi pad voltage vượt VDD+0.7V hoặc rơi xuống dưới VDD-0.7V; được tạo từ P+ diffusion trong n-well và N+ diffusion trong p-substrate
- **Resistor:** Giới hạn dòng điện và bảo vệ secondary protection; được tạo từ diffusion hoặc polysilicon

**Chuỗi bảo vệ:**

```
PAD → Primary ESD Elements → Current Limiting Resistor → Secondary ESD Elements → Internal Circuits
```

### 7.4 Block I/O Pin Placement Principles

Block Pin được đặt dựa trên hai tiêu chí chính:

- **Timing:** Minimize signal delay
- **Congestion:** Minimize routing difficulty

Thường được đặt ở **Metal 3 (M3)** với mục đích:

- Minimize signal delays
- Minimize crosstalks
- Grouping related signals together

### 7.5 Pad Configurations (Giải Pháp cho Pad-Limited Design)

|Cấu hình|Mô tả|
|---|---|
|**Inline IO Placement**|Đặt Pad thẳng hàng, tiêu chuẩn nhất|
|**Staggered Non-CUP IO**|Pad xen kẽ 2 hàng nhưng Bonding Pad ở ngoài I/O body|
|**Staggered CUP IO**|Circuit Under Pad — Bonding Pad nằm ngay trên I/O body, giảm diện tích die đáng kể|
|**Flip Chip IO**|Bump và driver cell đặt ở periphery hoặc core area; kết nối qua Redistribution Layer (RDL)|

### 7.6 Seal Ring và Scribe Line

**Seal Ring:**

- Cấu trúc bảo vệ bao quanh active die area
- Mục đích: Chống mechanical damage khi cắt wafer, chống moisture/contamination, chống electrical overstress
- Thành phần: Metal layer rộng chồng lên nhau, có thể có diffusion, well, poly, contact/via
- **Lưu ý:** Seal Ring liên tục có thể gây propagation của noise/signal không mong muốn → cần dùng Seal Ring có gaps

**Scribe Line (Saw Street/Scribe Street):**

- Vùng hẹp không có chức năng giữa các die trên wafer
- Dùng để wafer saw cắt chip ra khỏi wafer

---

## 8. Place Macros

### 8.1 Định Nghĩa Macro

Macro là **pre-designed, fixed-function block** được xử lý như hard IP trong quá trình place & route. Khác với synthesized logic được tạo từ Standard Cell, Macro **không linh hoạt** về kích thước, hình dạng hay cấu trúc bên trong.

**Phân loại Macro:**

|Loại|Ví dụ|
|---|---|
|Memory Macro|SRAM, ROM, Register File, NVM|
|Analog Block|ADC, DAC, PLL|
|Hard IP Block|USB PHY, SERDES|
|I/O Macro|Specialized Pad hoặc PHY ở vùng ngoại vi die|

### 8.2 Mục Tiêu của Macro Placement

Macro placement xác định vị trí tối ưu cho các block lớn để đạt PPA tốt nhất:

- **Optimize Area Utilization:** Minimize white space, layout compact
- **Minimize Wire Lengths:** Giảm interconnect delay và dynamic power
- **Facilitate Routing:** Tránh congestion bằng spacing và structured placement phù hợp
- **Shorten Timing-Critical Paths:** Minimize delay trên critical net
- **Reduce Switching Power:** Giảm dynamic power bằng cách đặt frequently-toggling block gần nhau
- **Mitigate EMIR Risks:** Ngăn hot spot, đảm bảo phân bố dòng điện đều cho IR drop và electromigration resilience

---

## 9. Bảy Yếu Tố Ảnh Hưởng Đến Macro Placement

### 9.1 Yếu Tố 1: Timing Closure

**Tại sao quan trọng:** Timing closure là một trong những mục tiêu khó nhất trong physical design. Macro placement có thể quyết định thành bại của critical path delay.

**Best Practices:**

- Group các Macro liên quan để minimize cross-region delay
- Ưu tiên timing-critical data path khi đặt Macro
- Tránh đặt Macro thường xuyên giao tiếp với nhau ở hai đầu đối lập của Core

**Common Pitfalls:**

- Bỏ qua hướng data flow → Macro timing-critical đặt quá xa nhau → wire delay dài, khó close timing
- Bỏ qua nhu cầu clock distribution → Clock sink Macro đặt xa clock source/buffer region → clock skew và latency quá cao
- Không reserve routing resource gần critical Macro → late-stage congestion → route detour → ảnh hưởng setup/hold timing

### 9.2 Yếu Tố 2: Power Consumption

**Tại sao quan trọng:** Macro switching thường xuyên tiêu thụ significant dynamic power. Interconnect dài + high-activity gây lãng phí năng lượng.

**Best Practices:**

- Đặt Macro giao tiếp thường xuyên gần nhau → giảm switching current trên long net
- Căn chỉnh Macro placement theo hướng data flow → tránh detour và long toggling path

**Common Pitfalls:**

- Spread high-activity Macro quá xa nhau → dynamic power quá cao
- Data path bị misalign → signal đi theo zig-zag hoặc loop → tăng switching net capacitance → tăng power
- Bỏ qua toggle rate khác nhau giữa các Macro → high-toggle block cần placement cẩn thận hơn low-activity block

### 9.3 Yếu Tố 3: System Interface

**Tại sao quan trọng:** Interface-related Macro cần kết nối sạch với thế giới bên ngoài hoặc các block cụ thể trên chip. Placement không tốt → route dài, vấn đề noise, khó integrate.

**Best Practices:**

- Đặt Macro gần I/O Pad hoặc interface logic liên quan (ví dụ: flash memory gần IO controller)
- Giữ Analog/RF Macro nhạy cảm xa vùng digital ồn ào hoặc IO region → thêm isolation ring → giảm noise coupling

**Common Pitfalls:**

- Interface Macro đặt xa IO → kết nối dài, congested, delay cao
- Bỏ qua nhu cầu isolation của sensitive block → lỗi chức năng hoặc signal integrity

### 9.4 Yếu Tố 4: Clock Distribution

**Tại sao quan trọng:** Macro có clocked interface (ví dụ SRAM) cần clock tree path cân bằng tốt. Placement không tốt → excessive skew, high insertion delay.

**Best Practices:**

- Đặt clocked Macro gần core clock distribution network → giảm clock skew, dễ CTS hơn

**Common Pitfalls:**

- Clocked Macro bị đặt cô lập → clock route dài → skew cao và insertion delay lớn

**Minh họa:**

```
BAD:  PLL → Clock Gen → CPU  ←→  (xa) SRAM  [CLK đi đường dài]
GOOD: PLL → Clock Gen → SRAM | CPU  [CLK đường ngắn, cân bằng]
```

### 9.5 Yếu Tố 5: Routing Congestion

**Tại sao quan trọng:** Macro chặn routing resource phía trên chúng. Đặt Macro quá sát → bóp nghẹt routing channel → route detour dài hơn → delay tăng → có thể routing fail.

**Best Practices:**

- Để đủ khoảng cách giữa các Macro → routing channel cho signal, clock, power
- Phân tán Macro có pin density cao → giảm congestion cục bộ
- Dùng early congestion analysis (trial routing hoặc congestion map) → phát hiện hotspot sớm, điều chỉnh vị trí Macro trước detailed placement

**Common Pitfalls:**

- Macro đặt quá sát → không đủ space cho signal và clock route
- High-pin Macro cluster trong một vùng → congestion cục bộ dày đặc
- Macro lớn chặn main routing channel → route detour → timing issue

### 9.6 Yếu Tố 6: Power Delivery & EM/Thermal

**Tại sao quan trọng:** Macro hút dòng điện lớn và phải kết nối hiệu quả với power grid. Placement không tốt → IR drop, electromigration (EM) failure, thermal hotspot. Các high-power Macro cluster với nhau → tăng local power density và nhiệt độ.

**Best Practices:**

- Đặt high-power Macro gần vùng power grid mạnh → giảm local IR drop
- Phân tán Macro có heavy current demand → tránh hotspot, giảm EM stress
- Để room cho power strap và tap cell → phân bố dòng và decoupling capacitance

**Common Pitfalls:**

- Cluster power-hungry Macro → IR drop và thermal issue cục bộ
- Đặt Macro xa power source → voltage droop và EM violation

**Nguyên tắc cụ thể:**

> Đặt block có performance cao nhất và tiêu thụ power lớn nhất: **gần power Pad** (giảm IR drop) và **xa nhau** (giảm EM).

Cần dành space để tapping VDD/VSS giữa các Macro.

### 9.7 Yếu Tố 7: Placement Rules & Foundry Requirements

**Tại sao quan trọng:** Foundry áp đặt placement rule để đảm bảo manufacturability, device reliability và layout integrity. Vi phạm có thể dẫn đến fab rejection hoặc phải rework sau layout.

**Best Practices:**

- **Fix Macro positions** trước khi finalize Floorplan → ngăn Macro bị di chuyển trong Placement và optimization
- **Insert Boundary Cell** theo yêu cầu → đảm bảo Cell ở boundary giữ nguyên behavior được characterize
- **Insert Tap Cell** theo yêu cầu → ngăn latch-up, đảm bảo well/substrate connection theo foundry rule
- **Duy trì poly orientation đồng nhất** nếu process yêu cầu → đặc biệt quan trọng với memory Macro (SRAM) ở advanced node
- **Giữ Macro đủ xa die edge** → tránh can thiệp với seal ring, tuân thủ dicing margin rule

---

## 10. Best Practices Tổng Hợp cho Macro Placement

### 10.1 Tối Ưu PPA Tổng Thể

**① Tạo Standard Cell placement area lớn và liên tục (contiguous)** → Kiểm soát congestion tốt hơn, nhiều routing & placement resource hơn → Ít delay, ít area cần cho timing fix, ít power

**② Để room cho routing & cell placement xuyên qua macro stack** → Long wire có thể được buffer → Nhiều routing option hơn, ít wiring detour hơn → Less delay → Better Performance

**③ Pin facing each other để minimize routing (khi hai Macro kết nối với nhau)** → Dây ngắn hơn → ít delay, ít area cho timing fix, ít power

**④ Pin facing outward nếu không kết nối với Macro kề bên**

**⑤ Pin của Macro tương tác nhiều với Standard Cell → hướng về phía SC area** → Dây ngắn hơn → Better Performance + Lower Power + Less Area

**⑥ Pins away from corners** → Tránh routing khó ở góc

### 10.2 Data Path Optimization

Cần xem xét **data flow diagram** hoặc high-level design description từ chip designer trước khi đặt Macro.

Ví dụ DSP chip:

- FFT operation thực hiện Multiply-Accumulate với Coefficients trên Data từ XRAM và YRAM, Coefficients từ CROM
- → XRAM/YRAM, CROM và PRAM (Program RAM) được đặt **gần nhau** để đảm bảo tốc độ và tiết kiệm power

**Tầm quan trọng:** $$\text{Giảm wirelength trên timing-critical path} \Rightarrow \begin{cases} \text{Less delay} \ \text{Less cell area for timing fix} \ \text{Less switching power} \end{cases} \Rightarrow \text{Better PPA}$$

### 10.3 Macro Orientation Analysis

**Phân tích connectivity để xác định orientation tốt nhất:**

- **Macro không kết nối với nhau:** Pin hướng ra ngoài (outward)
- **Macro kết nối với nhau:** Pin hướng vào nhau (facing each other) để minimize signal length

### 10.4 Flight Line Analysis

**Flight line** là công cụ trực quan hóa connectivity giữa các Macro và IO Port, giúp:

- Phát hiện potential routing congestion hotspot
- Xác định vị trí đặt Macro để minimize routing
- Phát hiện crisscross connectivity (cần tránh)

**Quy trình:**

1. Phân tích flight line từ Macro đến IO Port
2. Phân tích flight line từ Macro đến Macro
3. Đặt Macro gần IO Port tương ứng (boundary placement)
4. Đặt Macro có connectivity cao gần nhau

**Công thức xác định khoảng cách tối thiểu giữa hai Macro:**

$$d_{\min} = \frac{\text{Pitch of Routing Layers} \times \text{No. of Pins to be Routed}}{\text{Available Routing Layers in preferred direction}} + \text{Buffer Spacing}$$

### 10.5 Macro Placement cho Routability

**GOOD — Contiguous Standard Cell Placement Area:** Một khối rows rộng, không bị ngắt quãng cho cell placement.

**BAD — Các Pattern Cần Tránh:**

- **Chimney:** Vùng placement hẹp, không có cell placement area, wire density cao
- **Notch:** Góc lõm tạo ra high wire density
- **Fragmented region:** Vùng bị chia nhỏ bởi Macro hoặc boundary → congestion và suboptimal timing

> **Quy tắc:** High wire density = Congestion problems

---

## 11. Placement Blockages và Routing Blockages

### 11.1 Placement Blockage Types

**Hard Blockage:** Chặn hoàn toàn Placement trong vùng này.

**Partial Blockage:** Cho phép Placement đến một mức utilization tối đa nhất định (ví dụ 50%).

**Soft Blockage:** Chỉ cho phép một số Cell nhất định như Buffer, Inverter, Clock Gater.

**Macro-only Blockage:** Chặn Placement của Macro trong vùng này (Standard Cell vẫn được đặt).

**Halo:** Loại blockage đặc biệt xung quanh Macro:

- Kích thước blockage cố định
- Gắn liền với Macro cụ thể
- **Halo di chuyển cùng với Macro** khi Macro được di chuyển

### 11.2 Routing Blockages

Routing Blockage ngăn routing trong một vùng cho **các layer được chỉ định**. Congestion có thể được quản lý bằng **một trong hai cách**:

- Placement Blockage (gián tiếp: giảm cell → giảm wire)
- Routing Blockage (trực tiếp: cấm route trên layer cụ thể)

---

## 12. Tap Cells và Boundary Cells

### 12.1 Tap Cells

Có hai loại Standard Cell:

**Traditional Standard Cell:** Có tích hợp sẵn well tap bên trong cell body (n-well và p-substrate connection trực tiếp).

**Tapless Standard Cell:** Không có well tap tích hợp → cần Tap Cell riêng biệt.

**Tap Cell:** Cell chuyên dụng chỉ chứa well tap, không có logic. Được insert theo foundry rule để:

- Ngăn latch-up
- Đảm bảo well/substrate connection theo đúng quy định

Tap Cell thường được insert theo pattern stagger (so le) với khoảng cách nhất định (ví dụ 60μm).

### 12.2 Boundary Cells

Được insert tại boundary giữa các vùng để đảm bảo Cell ở ranh giới giữ nguyên electrical behavior đã được characterized. Có tầm quan trọng đặc biệt ở advanced node.

---

## 13. Mối Quan Hệ Giữa Các Khái Niệm — Tổng Hợp

```
FLOORPLANNING
│
├── DEFINE BOUNDARY
│   ├── Die Area = IO Ring + Core-to-IO Spacing + Core Area
│   ├── Core Area = f(SC Area, Macro Area, CTS overhead, Utilization)
│   ├── Aspect Ratio ↔ Routing Resource (H vs V tracks)
│   └── Block vs Full-chip boundary (inherited vs designer-defined)
│
├── PLACE I/O
│   ├── Full-chip Pad ↔ Package/PCB
│   │   ├── ESD Protection (Diode Clamp + Resistor)
│   │   ├── Power Supply Cell (VDD/VSS ring continuity)
│   │   ├── Corner Cell + Filler Cell
│   │   ├── Pad Config: Inline / Staggered / CUP / Flip-Chip
│   │   └── Seal Ring + Scribe Line
│   └── Block-level Pin ↔ other blocks
│       ├── Placement: Timing + Congestion
│       └── Typically M3
│
└── PLACE MACROS
    ├── Types: Memory / Analog / Hard IP / I/O Macro
    ├── Goals: PPA optimization
    ├── 7 Key Factors:
    │   ├── 1. Timing Closure → group related macros, data path direction
    │   ├── 2. Power → proximity of high-toggle macros
    │   ├── 3. System Interface → near IO, isolation rings
    │   ├── 4. Clock Distribution → near clock network
    │   ├── 5. Routing Congestion → spacing, flight line analysis
    │   ├── 6. Power Delivery/EM → near power grid, spread out
    │   └── 7. Foundry Rules → fix, boundary cell, tap cell, distance from edge
    ├── Tools: Flight Line Analysis, Placement Blockage, Halo, Routing Blockage
    └── Best Practices: contiguous SC area, pin orientation, data path alignment
```

---

## 14. Packaging và Bonding Wire Requirements

Khi thiết kế IO Pad placement cho full-chip, phải tuân thủ các ràng buộc bonding wire:

- **No Crossing:** Wire không được giao nhau
- **Minimum Spacing:** Khoảng cách tối thiểu giữa các wire
- **Maximum Angle:** Góc tối đa so với phương thẳng đứng
- **Maximum Length:** Chiều dài wire tối đa

**Lưu ý:** Pad có thể được di chuyển để giải quyết wire crossing violation. Nếu wire crossing xảy ra tại bond finger → vi phạm thiết kế.
