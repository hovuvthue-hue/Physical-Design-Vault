# Lecture 11 – Routing (Part I): Phân Tích Lý Thuyết Toàn Diện

## 1. Vị Trí Của Routing Trong IC Physical Design Flow

Routing là bước thứ năm trong chuỗi physical design flow theo thứ tự:

**Data Import → Floorplanning → Placement → CTS → Routing → Data Export**

Routing nhận đầu vào là database đã hoàn thành CTS (Clock Tree Synthesis) và tạo ra layout hoàn chỉnh sẵn sàng cho Data Export, sign-off, và tape-out.

---

## 2. Mục Đích Và Vai Trò Của Routing

Routing thực hiện các chức năng cốt lõi sau:

**Chức năng kết nối vật lý:**

- Tạo kết nối vật lý giữa các cell, macro, và IO pad thông qua dây kim loại (metal wire)
- Chuyển đổi kết quả placement thành layout có thể sản xuất được (manufacturable layout)

**Chức năng tối ưu hóa:**

- Thỏa mãn timing và physical design constraints
- Tối ưu hóa timing, congestion, và routing quality tổng thể
- Đảm bảo signal integrity và độ tin cậy (reliability)
- Đảm bảo khả năng sản xuất (manufacturability) và tuân thủ DRC

---

## 3. Tầm Quan Trọng Của Routing

Routing là giai đoạn quyết định cuối cùng về mặt vật lý vì nó:

- **Xác định kết nối vật lý cuối cùng** của toàn bộ thiết kế
- **Ảnh hưởng mạnh mẽ** đến timing, power, area, và signal integrity — bốn chỉ tiêu chất lượng thiết kế căn bản (PPA + SI)
- **Có thể tạo ra hoặc giải quyết** các vấn đề về congestion, DRC, và reliability
- **Đóng vai trò then chốt** trong việc đạt được timing closure sau placement và CTS, vì parasitic RC của wire ảnh hưởng trực tiếp đến delay
- **Ảnh hưởng đến parasitics, crosstalk, IR-drop, và EM** (electromigration) — các hiệu ứng vật lý quan trọng trong thiết kế hiện đại
- **Tạo ra topology layout cuối cùng** được sử dụng cho RC extraction, verification, và sign-off

---

## 4. Mục Tiêu Của Routing (Objectives)

Routing phải đồng thời cân bằng nhiều mục tiêu xung đột nhau, được biểu diễn dưới dạng mô hình "bánh xe" với LVS/DRC ở trung tâm và bốn chiều tỏa ra:

### 4.1 Mục Tiêu Chức Năng

- Hoàn thành **tất cả kết nối net** mà không có open (đứt mạch) hoặc short (ngắn mạch)
- Thỏa mãn DRC, LVS, và tất cả routing constraints

### 4.2 Mục Tiêu Timing

- Tối ưu hóa timing bằng cách kiểm soát wire delay, slew, và parasitics
- Mối liên hệ: wire dài hơn → resistance và capacitance cao hơn → delay lớn hơn → timing xấu hơn

### 4.3 Mục Tiêu Routing Quality

- Giảm congestion và cải thiện routability
- Tối thiểu hóa wirelength không cần thiết, số lượng via, và routing detour

### 4.4 Mục Tiêu Reliability & Manufacturability

- Duy trì signal integrity và độ tin cậy điện
- Cân bằng trade-off giữa power, performance, area, và manufacturability

---

## 5. Routing Inputs (Đầu Vào Cho Routing)

### 5.1 CTS Design Database

Database từ bước CTS_opt chứa đầy đủ:

- **Netlist**: Mô tả kết nối logic của thiết kế
- **Timing constraints** (SDC/MMMC): Xác định yêu cầu timing cho từng path
- **Physical constraints**: Bao gồm die size và IO placement

### 5.2 Technology Files (File Công Nghệ)

Hai loại file công nghệ quan trọng:

- **PnR Technology File** (thường là LEF): Định nghĩa design rules và routing layers
- **Interconnect Technology File** (ITF/ICT): Định nghĩa mô hình RC của dây và via

### 5.3 Library Files

- **Logical/Timing Library** (`.lib`): Chứa timing model của cell
- **Physical Library** (`.lef`): Chứa thông tin vật lý của cell (shape, pin location)

### 5.4 Special Net Requirements & Sign-off Requirements

- Các yêu cầu đặc biệt cho power/ground net và clock net
- Yêu cầu sign-off: DRC clean, timing clean, LVS clean trước tape-out

---

## 6. Technology Files — Phân Tích Chi Tiết

### 6.1 Vai Trò Của Technology Files

Technology files mô tả đặc tính vật lý và điện của technology node mục tiêu (ví dụ: TSMC 3nm). Đây là "ngôn ngữ" mà EDA tool dùng để hiểu quy trình sản xuất. Chúng được cung cấp bởi foundry (fab), không phải do designer tạo ra.

### 6.2 PnR Technology File

File này được sử dụng bởi PnR routing tool và thường được thể hiện dưới dạng **LEF (Library Exchange Format)**. Nội dung bao gồm:

**LEF File Info (Header):**

- Đơn vị đo lường (capacitance, current, voltage, frequency, database units)
- Các property definitions

**Routing Layers:** Mỗi layer được định nghĩa với:

- `TYPE ROUTING`: Xác định đây là routing layer
- `DIRECTION`: Hướng routing ưu tiên (HORIZONTAL hoặc VERTICAL)
- `PITCH`: Khoảng cách giữa các track
- `MINWIDTH`, `MAXWIDTH`: Chiều rộng tối thiểu và tối đa của wire
- Antenna model: Quy tắc về antenna effect

**Layout Design Rules:** Các rule quan trọng bao gồm:

- `SPACING`: Khoảng cách tối thiểu giữa hai wire trên cùng layer
- `MINSTEP`: Độ bước tối thiểu
- `ENCLOSURE`: Quy tắc bọc quanh via

**Standard Cell Heights:**

- `MANUFACTURINGGRID`: Grid sản xuất
- `SITE core`: Kích thước một site cell, xác định chiều cao standard cell

**Via Rules (Cut Layer):** Cho mỗi via layer (VIA1, VIA2...):

- `TYPE CUT`: Xác định đây là cut layer (via)
- `SPACING`, `ENCLOSURE BELOW/ABOVE`: Quy tắc spacing và enclosure

**Routing Rules:**

- `SPACINGTABLE`: Bảng spacing theo chiều dài và chiều rộng wire (parallel run length)
- `VIARULE VIAGEN`: Quy tắc tạo via tự động

**Routing Tracks:** Định nghĩa lưới routing:

```
TRACKS Y 0.0 DO 200 STEP 0.2 LAYER M1
TRACKS X 0.0 DO 100 STEP 0.2 LAYER M2
```

Ý nghĩa: Trên layer M1, tạo 200 track theo chiều Y, bắt đầu từ 0.0, bước 0.2μm.

### 6.3 Interconnect Technology File (ITF)

File ITF/ICT được sử dụng bởi **RC extraction tool**, không phải PnR tool trực tiếp. Nó cung cấp dữ liệu để tính toán parasitic RC của mỗi wire và via.

**Cấu trúc ITF:**

```
TECHNOLOGY "Generic_7nm"
UNIT
  RESISTANCE ohm
  CAPACITANCE fF
  LENGTH um
END UNIT
```

**Thông tin cho mỗi metal layer:**

- `WIDTH`, `THICKNESS`, `SPACING`: Tham số hình học
- `RESISTANCE`: Điện trở theo đơn vị chiều dài (Ω/μm)
- `CAPACITANCE`:
    - `LATERAL`: Capacitance theo chiều ngang (coupling với wire cùng layer)
    - `VERTICAL`: Capacitance theo chiều dọc (coupling với layer trên/dưới)
    - `FRINGE`: Fringe capacitance (hiệu ứng cạnh)

**Thông tin cho via:**

- `RESISTANCE`: Điện trở via ($\Omega$/contact)
- `ENCLOSURE`: Khoảng enclosure tối thiểu

**Process Corner:**

```
CORNER RC_Typ
  TEMPERATURE 25
  VOLTAGE 0.8
END CORNER
```

Mỗi ITF file tương ứng với **một process corner cụ thể** (Typical/Fast/Slow × nhiệt độ × điện áp). RC extraction tool cần nhiều ITF files để cover tất cả corners.

**Flow RC Extraction:**

$$\text{Fabrication} \xrightarrow{\text{ITF}} \text{PnR} \xrightarrow{\text{SPEF}} \text{RC Extraction} \xrightarrow{\text{SPEF}} \text{STA}$$

Foundry cung cấp ITF, PnR tool tạo ra routing geometry (SPEF sơ bộ), RC extraction tool tính SPEF chính xác, STA tool sử dụng SPEF để timing analysis.

---

## 7. Metal Stack (Cấu Trúc Kim Loại Nhiều Lớp)

### 7.1 Định Nghĩa

Metal stack là tập hợp nhiều lớp kim loại (metal layer) được ngăn cách bởi vật liệu điện môi (dielectric/insulating material) và được kết nối theo chiều dọc thông qua via. Đây là "kiến trúc" của hệ thống kết nối trong chip.

### 7.2 Đặc Điểm Thiết Kế

Mỗi metal layer được thiết kế với:

- **Độ dày (thickness)** cụ thể
- **Chiều rộng (width)** đặc trưng
- **Điện trở suất (resistivity)** phù hợp với mục đích sử dụng

### 7.3 Phân Loại Layer Theo Quy Ước TSMC

|Ký hiệu|Loại Layer|Mục đích|Đặc điểm|
|---|---|---|---|
|M1 (Local)|Local routing layer|Routing bên trong standard cell|Wire nhỏ nhất, mật độ cao nhất|
|x (standard-thickness)|Dense signal routing|Kết nối inter-block|Cân bằng performance & routing density|
|y (thicker)|Semi-global routing|Kết nối tầm trung|Điện trở thấp hơn x|
|z (even-thicker)|Global signals, clock|Phân phối clock, tín hiệu quan trọng|Delay thấp hơn nữa|
|u (ultra-thick)|Power distribution|Lưới power (PDN)|Điện trở rất thấp, dòng điện lớn|
|r (redistribution)|I/O connections|Bump, bonding pad|Không dùng cho signal routing|

**Ví dụ naming convention TSMC:**

- `5x1z1u`: 1 local layer (M1) + 5 standard-thickness + 1 even-thicker + 1 ultra-thick
- `5x2y2z`: 1 local layer (M1) + 5x + 2y + 2z

### 7.4 Ví dụ Metal Stack TSMC 22nm (5x1z1u)

|Layer|Name|Type|Purpose|Notes|
|---|---|---|---|---|
|M1|Metal 1|Local Routing|Inside standard cells|Smallest wires, high density|
|M2–M6 (5x)|Metal 2–6|Intermediate Routing|Inter-block signal routing|Balanced performance & routing resources|
|M7 (1z)|Metal 7|Thick Metal|Clock nets, wide signals|Lower resistance, EM safe|
|M8 (1u)|Metal 8|Ultra-Thick Metal|Power grid, global signals|Designed for high current|
|AP|Aluminum Pad|Bonding Pad Layer|I/O pads, bump connections, ESD protection|Not for routing, just pads|

### 7.5 Trade-off: High Performance vs. High Density

Metal stack được tối ưu theo hai hướng khác nhau:

**High Performance (CPU-oriented):**

- Wire rộng hơn → điện trở thấp hơn → delay thấp hơn
- Spacing lớn hơn → capacitance thấp hơn, noise thấp hơn
- Nhiều layer hơn với pitch lớn hơn (4× pitch)
- Ưu tiên tốc độ, chấp nhận diện tích lớn hơn

**High Density (SoC-oriented):**

- Wire hẹp hơn, spacing nhỏ hơn
- Pitch nhỏ hơn (1× pitch ở layer thấp, 2-6 layers cho dense routing)
- Ưu tiên diện tích nhỏ và chi phí sản xuất thấp
- Chấp nhận performance thấp hơn

Mối quan hệ định lượng:

$$R_{wire} = \rho \cdot \frac{L}{W \cdot T}$$

$$C_{wire} \approx C_{lateral} + C_{vertical} + C_{fringe}$$

$$\text{Delay} \propto R \cdot C$$

Trong đó $\rho$ là điện trở suất, $L$ là chiều dài, $W$ là chiều rộng, $T$ là độ dày.

---

## 8. Routing Tracks

### 8.1 Định Nghĩa

Routing track là các đường đi được định nghĩa trước (predefined paths) mà router sử dụng để kết nối các net giữa các cell. Track được sắp xếp theo chiều ngang và dọc trên mỗi routing layer.

### 8.2 Pitch

**Pitch** là khoảng cách giữa hai routing track kề nhau. Pitch nhỏ hơn → mật độ routing cao hơn nhưng nguy cơ DRC violation cao hơn.

$$\text{Pitch} = \text{Wire Width} + \text{Wire Spacing}$$

### 8.3 Cell Height Tính Theo Track

"Cell height in tracks" là số lượng horizontal M1 routing track vừa khít trong một standard cell row. Ví dụ:

- **7-track cell**: Chiều cao cell = 7 M1 track pitches
- Track count thấp hơn → cell nhỏ hơn → mật độ cao hơn nhưng routing khó hơn

Trong standard cell layout:

- **W3**: Horizontal grid (track ngang)
- **W2**: Vertical grid (track dọc)
- **W4, W1, H**: Các thông số khác liên quan đến cell boundary và power rail

### 8.4 Grid-Based vs. Gridless Routing

**Grid-Based Routing:**

- Wire chỉ được đặt tại các giao điểm của routing grid
- Đơn giản hơn, nhanh hơn
- Đôi khi không tối ưu về diện tích
- Phổ biến hơn trong standard cell design

**Gridless Routing:**

- Wire có thể được đặt tự do, không bị ràng buộc bởi grid
- Linh hoạt hơn, có thể tối ưu hơn
- Phức tạp hơn về mặt thuật toán
- Phù hợp cho analog hay custom design

---

## 9. Routing Metal Layers và Vias — Nguyên Lý Kết Nối Đa Lớp

### 9.1 Số Lượng Layer Trong Thiết Kế Hiện Đại

Thiết kế hiện đại sử dụng từ 3 đến 19 metal layer để routing. Số lượng layer tăng theo technology node tiên tiến hơn vì mật độ cell tăng, cần nhiều routing resources hơn.

### 9.2 Kết Nối Giữa Các Layer Qua Via

Mỗi via kết nối hai metal layer liền kề:

- **VIA1** (hoặc V1): Kết nối M1 và M2
- **VIA2** (hoặc V2): Kết nối M2 và M3
- ... và tương tự

Tín hiệu di chuyển theo chiều dọc qua các via này.

### 9.3 Preferred Routing Directions

Mỗi metal layer có một **hướng routing ưu tiên**, và các layer kề nhau có hướng vuông góc:

|Layer|Hướng Ưu Tiên|
|---|---|
|M1 (Blue)|Horizontal|
|M2 (Yellow)|Vertical|
|M3 (Red)|Horizontal|
|M4|Vertical|
|M5|Horizontal|
|M6|Vertical|

**Lý do vật lý:** Hướng xen kẽ này giảm coupling capacitance giữa các wire song song trên các layer kề nhau. Nếu M1 và M2 đều horizontal, các wire song song sẽ tạo coupling capacitance lớn, gây crosstalk nghiêm trọng.

**Lợi ích thực tiễn:**

- Giảm congestion vì routing phân bổ đều theo cả hai chiều
- Giảm coupling noise giữa wire trên các layer kề nhau
- Cải thiện khả năng sản xuất (manufacturability)

### 9.4 Wire Pitch, Width, và Via Spacing

Ở góc nhìn cross-section:

- **Wire width**: Chiều rộng của metal wire
- **Wire spacing**: Khoảng cách giữa hai wire kề nhau trên cùng layer
- **Via width**: Chiều rộng của via cut
- **Via spacing**: Khoảng cách giữa hai via kề nhau

Tất cả các thông số này được quy định chặt chẽ trong technology file và phải tuân thủ DRC.

---

## 10. Những Quyết Định Của Router Về Metal

### 10.1 Sáu Mục Tiêu Mâu Thuẫn

Mọi quyết định metal của router đều phải xem xét đồng thời sáu mục tiêu, thường xung đột với nhau:

```
              Congestion
                  |
    Timing ——— Metal ——— SI/Noise
                  |
    Reliability  Area  Power
```

### 10.2 Layer Assignment và Routing Direction

**Lower metals (M1, M2...):** Short local cell routing — kết nối pin trong phạm vi gần **Upper metals (M5, M6...):** Longer routes / critical nets / clocks — kết nối tầm xa

**Lý do:** Metal layer trên cao hơn có điện trở thấp hơn (wire dày hơn) và capacitance thấp hơn (xa substrate và các layer dưới hơn). Do đó, critical net và clock net ưu tiên routing trên layer cao để giảm delay.

### 10.3 Wire Width, Spacing, và NDR

**Default routing rule:** Wire width và spacing tối thiểu theo DRC — dùng cho hầu hết các signal net thông thường.

**Non-Default Rule (NDR):** Wire rộng hơn + spacing lớn hơn — dùng cho:

- Clock net (giảm coupling, cải thiện slew)
- Critical path (giảm delay và crosstalk)
- High-current net (giảm EM risk)
- Noisy net (cách ly noise)

Tác động của NDR:

- Wire rộng hơn → $R_{wire}$ giảm → delay giảm, nhưng $C_{wire}$ tăng → tradeoff
- Spacing lớn hơn → Coupling capacitance giảm → SI cải thiện, nhưng chiếm nhiều routing resource hơn

**Shielding:** Đặt wire GND (hoặc VDD) song song hai bên victim net:

- **Same-layer shielding**: GND — Signal — GND trên cùng layer
- **Adjacent-layer shielding**: Power/GND plane trên layer kề để che chắn signal

### 10.4 Layer Changes và Via Count

**Ít layer changes (few layer changes):**

- Đường đi ngắn hơn, ít via hơn
- RC thấp hơn → timing tốt hơn
- DRC risk thấp hơn

**Nhiều layer changes (many layer changes):**

- Đường đi linh hoạt hơn để tránh congestion hoặc obstacle
- Mỗi via thêm điện trở và capacitance
- RC cao hơn → timing xấu hơn, IR-drop tăng

Quan hệ via resistance:

$$R_{via,total} = \frac{R_{via,single}}{N_{cuts}}$$

Trong đó $N_{cuts}$ là số cut song song. Do đó multi-cut via có điện trở thấp hơn single-cut via.

---

## 11. Những Quyết Định Của Router Về Via

### 11.1 Layer Transition

Router quyết định khi nào một net cần chuyển từ layer này sang layer khác. Lý do chuyển layer:

- **Routing obstacle**: Có blockage hoặc wire khác chắn đường
- **Congestion**: Layer hiện tại quá tắc nghẽn
- **Direction rule**: Layer hiện tại không có hướng routing phù hợp với path cần đi

### 11.2 Via Types và Selection

|Loại Via|Mô Tả|Đặc Điểm|
|---|---|---|
|Single-cut via|1 cut (contact)|Điện trở cao, đơn giản, chiếm ít diện tích|
|2-bar-cut via|2 cut hình thanh|Điện trở thấp hơn, yield tốt hơn|
|3-square-cut via|3 cut hình vuông|Điện trở thấp hơn nữa|
|6-square-cut via|6 cut hình vuông|Điện trở rất thấp, reliability cao|
|Stack via|Via xếp chồng qua nhiều layer|Kết nối đa layer, tiết kiệm không gian ngang|
|Redundant via|Nhiều via song song cùng vị trí|Cải thiện yield và EM margin|

**Nguyên tắc:** Càng nhiều cut song song → $R_{via}$ càng thấp → timing tốt hơn, EM margin tốt hơn, yield cao hơn.

### 11.3 DRC Requirements Cho Via

**Top-view geometry checks:**

- **Via enclosure**: Metal layer phải "bọc" quanh via cut đủ rộng theo yêu cầu
- **Cut spacing**: Khoảng cách giữa hai cut trong cùng via structure phải đủ lớn

**Side-view layer connection:**

- Via phải kết nối đúng hai layer liền kề (ví dụ: VIA1 kết nối M1 và M2, không được kết nối M1 và M3 trực tiếp mà không qua M2)
- Đây là nguyên tắc stack via: muốn kết nối M1 và M3 cần stacked via: VIA1 + M2 + VIA2

### 11.4 Via Count Optimization

**Trên timing-critical net:**

- Giảm số via không cần thiết → giảm $R_{via}$ và $C_{via}$ → cải thiện timing

**Trên power/reliability-critical net:**

- Thêm via mạnh hơn → tăng khả năng chịu dòng điện → giảm EM risk

**Clock net:**

- Ít via → minimize RC → minimize delay và skew variation

**Power net:**

- Nhiều via với multi-cut structure → high current capacity

**Trade-off tổng quát:**

- Nhiều via hơn → điện trở thấp hơn, reliability tốt hơn → nhưng diện tích lớn hơn, congestion cao hơn, DRC phức tạp hơn

### 11.5 Post-Route Via Improvement

Sau khi routing hoàn thành, một bước tối ưu hóa via được thực hiện:

- **Single-cut via trên critical net** → swap sang **redundant (multi-cut) via** nếu không gian cho phép
- Cải thiện via array trong quá trình DFM optimization
- Đảm bảo kết quả vẫn DRC clean sau khi swap

---

## 12. Routing Blockages

### 12.1 Định Nghĩa và Mục Đích

Routing blockage là vùng địa lý trên chip mà router được chỉ định **không được route** (hoặc hạn chế route) qua. Đây là công cụ để kiểm soát hành vi của router.

**Mục đích sử dụng blockage:**

- Bảo vệ vùng dưới macro và IP block khỏi bị route qua
- Cách ly vùng analog khỏi noise từ digital routing
- Kiểm soát congestion trong vùng bị tắc nghẽn
- Ngăn wire đi vào vùng không được phép về DRC hoặc SI

### 12.2 Hard Blockage

**Định nghĩa:** Hoàn toàn ngăn cản routing bên trong vùng bị block.

**Hành vi của router:** Wire phải tìm đường vòng (detour) xung quanh vùng hard blockage. Không có exception nào.

**Khi nào dùng:**

- Bên dưới macro/IP block (router không được chui vào trong macro)
- Vùng cấm tuyệt đối theo thiết kế (ví dụ: vùng substrate tap, analog guard ring)
- Vùng mà bất kỳ wire nào đi qua cũng sẽ gây DRC hoặc functional failure

### 12.3 Soft Blockage

**Định nghĩa:** Ngăn cản routing trong vùng nhưng **cho phép nếu router không còn lựa chọn nào khác**.

**Hành vi của router:** Router ưu tiên tránh vùng soft blockage, nhưng nếu không có đường nào khác, có thể cho wire đi qua.

**Khi nào dùng:**

- Vùng không mong muốn có routing nhưng không cấm tuyệt đối
- Gần macro nhưng không phải bên trong
- Vùng congestion trung bình — muốn router giảm tải nhưng không ép buộc

### 12.4 Đặc Điểm Kỹ Thuật

- Blockage có thể áp dụng cho **specific metal layer** hoặc **tất cả routing layer**
- Ví dụ: "Block M1 và M2 trong vùng X, nhưng cho phép M3 trở lên đi qua"
- Cải thiện routing quality bằng cách giữ vùng nhạy cảm sạch sẽ và có kiểm soát

---

## 13. Routing Constraints

### 13.1 Khái Niệm

Routing constraint là tập hợp các quy tắc và hướng dẫn mà router phải tuân theo khi routing một net, một net class, hoặc một vùng cụ thể. Mục đích là đảm bảo thiết kế đáp ứng timing, SI, reliability, và manufacturability goals.

### 13.2 Phạm Vi Áp Dụng

- **Individual net**: Constraint chỉ áp dụng cho một net cụ thể (ví dụ: clock net)
- **Net class**: Constraint áp dụng cho một nhóm net (ví dụ: tất cả reset net, tất cả high-speed net)
- **Region**: Constraint áp dụng cho vùng địa lý trên chip

### 13.3 Loại Routing Constraints

**Critical net constraint (chiều rộng wire / preferred layer):**

- Wire rộng hơn mức default để giảm resistance
- Được route trên layer cao hơn (ít congested hơn, điện trở thấp hơn)

**Shielding constraint:**

- Bảo vệ noise-sensitive net bằng ground shield wire song song

**Layer / Region rule:**

- Tránh route qua macro hoặc IP block
- Chỉ cho phép routing trên specific layer trong vùng nhất định

**Priority-based routing:** Router route các net theo thứ tự ưu tiên:

1. Critical net (M6, upper layer) → route trực tiếp, ít detour
2. Signal net (M4, mid layer) → route sau, có thể detour quanh congested area
3. Local net (M2, lower layer) → route cuối, dùng space còn lại

### 13.4 Phương Pháp Thiết Lập

Constraints có thể được thiết lập qua:

- **Tcl constraints**: Câu lệnh trong script routing
- **Routing attributes**: Thuộc tính gắn vào net hoặc cell
- **NDR (Non-Default Rule)**: Quy tắc wire width/spacing không theo mặc định
- **DEF**: Định nghĩa trong Design Exchange Format file
- **Technology rules**: Quy tắc trong LEF/technology file

---

## 14. Crosstalk Effect — Hiệu Ứng Nhiễu Xuyên

### 14.1 Định Nghĩa Vật Lý

Crosstalk là sự coupling điện không mong muốn giữa hai wire (net) kề nhau. Nguyên nhân chính là **capacitive coupling** — khi hai wire chạy song song gần nhau, chúng tạo thành một tụ điện ký sinh (coupling capacitance $C_m$).

### 14.2 Mô Hình Aggressor–Victim

```
Aggressor net A ─────[Buffer]──────────────── [Buffer]─────
                                   Cm (coupling)
Victim net V   ─────[Buffer]──────────────── [Buffer]─────
                          C1 (to ground)  C2 (to ground)
```

- **Aggressor net**: Net đang switching (chuyển trạng thái), inject noise vào victim
- **Victim net**: Net bị ảnh hưởng bởi noise từ aggressor
- **Coupling capacitance $C_m$**: Tụ điện ký sinh giữa hai wire kề nhau

### 14.3 Ba Loại Ảnh Hưởng Của Crosstalk

**1. Speed up victim (tăng tốc):**

- Xảy ra khi aggressor và victim switching cùng chiều (cùng 0→1 hoặc 1→0)
- $C_m$ "trợ lực" cho victim, làm victim chuyển nhanh hơn
- Hệ quả: **Hold violation** vì dữ liệu đến sớm hơn dự kiến tại flip-flop thu

**2. Slow down victim (làm chậm):**

- Xảy ra khi aggressor và victim switching ngược chiều
- $C_m$ cản trở chuyển đổi của victim
- Hệ quả: **Setup violation** vì dữ liệu đến muộn hơn required time

**3. Glitch on victim (tạo nhiễu xung):**

- Xảy ra khi victim đang ở trạng thái static (0 hoặc 1 cố định)
- Aggressor switching gây ra spike/bump trên victim
- Nếu spike đủ lớn và vượt ngưỡng logic → **functional failure** (glitch được capture bởi flip-flop)
- Hai trường hợp: victim đang giữ 0 bị kéo lên → glitch cao; victim đang giữ 1 bị kéo xuống → glitch thấp

### 14.4 Điều Kiện Làm Crosstalk Tệ Hơn

**Tỷ lệ coupling:**

$$\frac{C_m}{C_m + C_1} \text{ (hoặc } \frac{C_m}{C_m + C_2}\text{)}$$

Tỷ lệ này càng cao → crosstalk effect càng mạnh.

**Các yếu tố làm tỷ lệ này tăng:**

- **Wire chạy song song dài hơn** → $C_m$ tăng tỷ lệ thuận với chiều dài song song
- **Wire khoảng cách gần hơn** → $C_m$ tăng theo nghịch khoảng cách
- **Wire trên layer cao hơn** → wire dày hơn nhưng spacing nhỏ hơn ở advanced node
- **Wire rộng hơn** (paradoxically) → diện tích mặt bên lớn hơn → $C_m$ tăng

### 14.5 Giới Hạn Noise Margin

Glitch nguy hiểm khi vượt qua vùng "Undefined Region" trong transfer characteristic của CMOS:

- $V_{OL}$ đến $V_{IL}$: vùng logic 0 (safe)
- $V_{IH}$ đến $V_{OH}$: vùng logic 1 (safe)
- $V_{IL}$ đến $V_{IH}$: **Undefined Region** — nếu tín hiệu rơi vào đây, output không xác định

$$NM_L = V_{IL} - V_{OL} \quad \text{(noise margin for logic 0)}$$ $$NM_H = V_{OH} - V_{IH} \quad \text{(noise margin for logic 1)}$$

---

## 15. Crosstalk-Aware Routing — Kỹ Thuật Giảm Thiểu Crosstalk

### 15.1 Shielding (Che Chắn)

**Same-Layer Shielding:** Đặt wire GND (ground) và wire Power song song hai bên signal wire trên cùng layer:

```
Power  │  Signal  │  Ground
```

Hiệu ứng: Wire shield được kết nối với supply (điện áp cố định, không switching), nên không inject noise vào signal. Đồng thời, shield "hấp thụ" coupling từ các aggressor bên ngoài trước khi chúng reach được victim.

**Adjacent-Layer Shielding:** Metal shield plane (thường là ground) trên layer liền kề (trên hoặc dưới) signal layer:

```
M2 (GND plane)
M1 (Signal)
Poly
```

Hiệu ứng: Giảm coupling theo chiều dọc giữa signal và các wire khác.

**Ứng dụng chính của shielding:**

- Clock net: Critical về timing, cần routing sạch nhất có thể
- Analog signal: Cực kỳ nhạy cảm với noise
- Critical path: Để đảm bảo timing không bị crosstalk shift

### 15.2 Segregation (Phân Vùng)

Tách biệt vùng "noisy" (nhiều switching activity) khỏi vùng "quiet" (noise-sensitive):

- **Noisy region**: Vùng có nhiều high-switching digital logic, clock tree, I/O buffers
- **Quiet region**: Vùng analog, low-power domain, timing-critical path

Cách thực hiện:

- Routing blockage: Ngăn wire từ noisy region đi qua quiet region
- Layer constraint: Noisy net chỉ được route trên certain layers
- Region rule: Quy tắc routing khác nhau cho mỗi vùng

### 15.3 Wire Spacing (Tăng Khoảng Cách)

Tăng spacing giữa wire song song vượt quá yêu cầu DRC tối thiểu:

$$C_m \propto \frac{1}{d}$$

Trong đó $d$ là khoảng cách giữa hai wire. Spacing lớn hơn → $C_m$ giảm → crosstalk giảm.

Cách thực hiện: Áp dụng **NDR** với spacing rule lớn hơn cho noise-sensitive net.

### 15.4 Net Ordering (Sắp Xếp Thứ Tự Net)

Sắp xếp các net sao cho:

- Noise-sensitive net (**victim**) được cách ly khỏi high-switching net (**aggressor**)
- Các net có switching activity giống nhau được nhóm lại gần nhau (giảm worst-case coupling)

Lợi ích: Giảm coupling tệ nhất mà không cần thêm routing resource.

### 15.5 Workflow Crosstalk-Aware Routing

1. **Trước khi routing**: Enable SI-driven routing options trong tool
2. **Trong khi routing**: Áp dụng routing rules cho critical net (extra spacing, shielding, preferred layers), dùng NDR cho clock/reset/high-fanout/critical path
3. **Sau khi routing**: Run SI-aware timing và noise analysis để verify

---

## 16. Routing Flow — Luồng Thực Hiện Routing

Routing flow gồm 5 giai đoạn tuần tự:

```
Setup → Global Routing → Detailed Routing → Post-Route Optimization → Chip Finishing & Signoff
```

---

## 17. Setup Phase — Chuẩn Bị Trước Routing

### 17.1 Kiểm Tra Đầu Vào

Trước khi chạy routing, phải kiểm tra chất lượng database từ CTS:

- **Physical clean**: Không có placement violation, không có overlap
- **Timing clean**: Không có quá nhiều violation từ CTS stage — các violation còn lại phải trong phạm vi mà routing + post-route optimization có thể recover

Nếu timing quá xấu từ CTS, routing sẽ không thể cứu được → cần quay lại placement hoặc CTS.

### 17.2 Timing Strategy

**Path grouping** — phân loại timing path thành các nhóm để tool tối ưu hóa có mục tiêu:

- `reset_path_groups`: Xóa phân nhóm cũ
- `create_basic_path_group -expanded`: Tạo nhóm cơ bản mở rộng (xem Section 22)

### 17.3 Routing Strategy

Các thiết lập routing strategy bao gồm:

**Layer configuration:**

- `design_top_routing_layer`: Layer cao nhất được phép route signal
- `design_bottom_routing_layer`: Layer thấp nhất được phép route signal (thường M2, vì M1 được dùng bởi cell trong LEF)

**Driven routing:**

- **Timing-driven routing**: Router ưu tiên path có critical timing → chọn đường ngắn hơn, layer tốt hơn cho critical net
- **SI-driven routing**: Router xem xét coupling capacitance → tránh chạy song song dài với aggressor

**Antenna fixing:**

- **Antenna effect**: Trong quá trình fabrication, metal dài bị tích điện do plasma etching → có thể phá hủy gate oxide
- `route_detail_fix_antenna true`: Router tự động insert antenna diode hoặc wire jogging để fix
- `route_fix_clock_nets true`: Cho phép router sửa clock net nếu cần

**Special net pre-routing:** `route_special` — route power (VDD/VSS) và các special net trước signal routing. Đây là bước quan trọng vì power net cần được đảm bảo kết nối trước khi signal routing bắt đầu.

### 17.4 Additional Routing Strategies

**Antenna diode insertion:**

- Antenna diode là cell đặc biệt có diode kết nối giữa pin và ground
- Diode discharge điện tích tích lũy trên wire dài trong quá trình fabrication
- `route_antenna_diode_insertion true`: Tự động insert diode khi cần

**Multi-cut via strategy:**

- `route_reserve_space_for_multi_cut true`: Dành chỗ sẵn cho multi-cut via
- `route_detail_use_multi_cut_via_effort high`: Tích cực sử dụng multi-cut via để cải thiện reliability

**Litho-driven routing:**

- `route_with_litho_driven true`: Router xem xét khả năng in ảnh hưởng (lithography-friendly patterns)
- Tránh tạo các pattern khó in (ví dụ: góc 45°, notch quá nhỏ)

**Wire spreading/widening:**

- `route_detail_post_route_spread_wire true`: Phân tán wire để tăng spacing
- `route_detail_post_route_wire_widen true`: Mở rộng wire nơi có thêm không gian — cải thiện EM margin và resistance

---

## 18. Global Routing — Lập Kế Hoạch Routing

### 18.1 Mục Đích Và Nguyên Lý

Global routing là **giai đoạn lập kế hoạch**, không tạo ra wire thực (actual legal wires/vias). Nó phân tích routing resources và lên kế hoạch đường đi cho từng net trước khi detailed routing thực hiện.

Analogy: Global routing giống như lập kế hoạch tuyến đường trên bản đồ thành phố (đường nào, khu nào) trước khi thực sự lái xe theo từng con đường cụ thể (detailed routing).

### 18.2 GCELL — Global Routing Cell

**GCELL** (Global Cell) là đơn vị cơ bản của global routing:

- Chip được chia thành lưới các GCELL (mỗi GCELL là một hình chữ nhật)
- Mỗi GCELL chứa nhiều routing track
- Mỗi **GCELL edge** (cạnh giữa hai GCELL kề nhau) có **routing capacity** giới hạn

**Routing capacity** của một GCELL edge = số routing track đi qua cạnh đó.

**Routing demand** = số net cần đi qua cạnh đó.

$$\text{Overflow} = \max(0, \text{Demand} - \text{Capacity})$$

Nếu overflow > 0 → congestion hotspot → cần reroute để giảm.

### 18.3 Các Bước Trong Global Routing

1. **Partition**: Chia thiết kế thành lưới GCELL
2. **Capacity estimation**: Ước tính routing capacity mỗi GCELL edge (dựa trên metal layer, track count, blockage)
3. **Topology planning**: Xây dựng topology gần đúng cho mỗi net (Steiner tree hay maze routing ở mức coarse)
4. **Layer assignment**: Chỉ định layer sơ bộ cho mỗi đoạn wire
5. **Demand allocation**: Phân bổ routing demand cho horizontal và vertical GCELL edges
6. **Congestion detection**: Xác định GCELL edge có overflow
7. **Path adjustment**: Điều chỉnh đường đi để giảm overflow → reroute từ GCELL hotspot sang đường đi khác ít tắc nghẽn hơn
8. **Routing guide generation**: Tạo routing guide cho detailed router

**Visualization:** GCELL edge được hiển thị màu sắc theo mức tải:

- Xanh (low demand): Thoải mái, nhiều resource trống
- Vàng (medium demand): Bình thường
- Đỏ (high demand / overflow risk): Nguy cơ tắc nghẽn, cần giải quyết

### 18.4 Thông Tin Global Routing Tính Đến

- Timing (ưu tiên đường ngắn cho critical net)
- Congestion (tránh GCELL quá tải)
- Blockage (tránh vùng không được route)
- Preferred directions (tôn trọng hướng routing của từng layer)
- Routing constraints (NDR, shielding, via rules)

### 18.5 Đầu Ra Của Global Routing

**Routing guide** — "bản đồ chỉ đường" cho detailed router, chỉ định:

- Vùng GCELL nào net này được phép đi qua
- Layer nào được ưu tiên cho từng đoạn

Detailed router sẽ tạo ra exact wires and vias bên trong các GCELL guide đó.

---

## 19. Detailed Routing — Routing Chi Tiết

### 19.1 Mục Đích

Detailed routing nhận routing guide từ global routing và **tạo ra wire và via thực tế** — các hình dạng metal (metal shape) cụ thể có thể được kiểm tra bằng DRC và được fabricate.

### 19.2 Nguyên Lý Hoạt Động

**Từ GCELL guide → exact wires:**

- Guide chỉ định vùng rộng (GCELL granularity)
- Detailed router xác định chính xác track nào được dùng, width bao nhiêu, via đặt ở đâu

**Pin access:**

- Router phải tìm cách tiếp cận pin của cell/macro
- Pin trên M1 (trong standard cell) → router cần via từ M2 xuống M1 để kết nối, hoặc route trực tiếp trên M1

**Rip-up and Reroute:**

- Trong quá trình routing, router có thể tạm thời chấp nhận violation (short, spacing violation) để đảm bảo connectivity
- Sau đó, rip-up (xóa) phần route đó và reroute theo hướng khác để fix violation
- Đây là iterative process

**Local optimization:**

- Giảm via count khi có thể
- Cải thiện wire length
- Tối ưu SI cho local segment

### 19.3 Đầu Ra Của Detailed Routing

**Routed database** — toàn bộ wire và via được định nghĩa chính xác:

- Được dùng cho DRC check
- Được dùng cho RC extraction (tính parasitic RC thực tế)
- Được dùng cho timing analysis sau route
- Được dùng cho post-route optimization

---

## 20. Post-Route Optimization

### 20.1 Tại Sao Cần Post-Route Optimization?

Trước routing, timing analysis dùng **estimated** parasitic RC (ước tính dựa trên placement, không có wire thực). Sau routing, **actual** parasitic RC được extract từ wire và via thực → timing có thể thay đổi đáng kể so với pre-route.

**Nguyên nhân timing degradation sau routing:**

- Wire dài hơn mong đợi (phải đi vòng vì congestion)
- Nhiều via hơn dự kiến
- Coupling capacitance từ wire kề nhau
- SI/noise effect từ crosstalk

### 20.2 Standard Post-Route Optimization (Part 1)

Chạy ngay sau khi detailed routing hoàn thành, sử dụng extracted post-route parasitics:

**Nội dung tối ưu hóa:**

- Fix remaining **setup violation**: Giảm delay trên critical path (buffering, resizing, routing cleanup)
- Fix remaining **hold violation**: Thêm delay trên fast path (insert buffer/delay cell)
- Improve **transition/slew**: Resize driver hoặc fix wire với slew quá xấu
- Reduce **capacitance**: Tách net fanout lớn
- Fix **SI/noise**: Cải thiện spacing, thêm shielding nếu cần
- Fix **EM**: Widen wire trên high-current path

**Phương pháp:**

- Local buffering: Insert buffer để chia wire dài
- Cell resizing: Tăng/giảm drive strength
- Routing cleanup: Route lại một số segment để giảm delay

### 20.3 Iterative Timing Closure Loop (Part 2)

Với thiết kế phức tạp, một vòng lặp iterative được thực hiện:

```
route_design → check timing/DRC → opt_design → route_opt → check → (lặp nếu cần)
```

Điều kiện dừng:

- Tất cả violation được giải quyết (timing clean + DRC clean), **hoặc**
- Không có cải thiện thêm (convergence)

---

## 21. QoR Checks Sau Baseline Routing

### 21.1 Timing Check

Kiểm tra WNS và TNS cho cả setup và hold:

$$\text{Slack} = T_{required} - T_{arrival}$$

- **WNS (Worst Negative Slack)**: Slack âm tệ nhất → mức độ nghiêm trọng của timing violation đơn lẻ tệ nhất
- **TNS (Total Negative Slack)**: Tổng tất cả slack âm → tổng "số nợ" timing cần giải quyết

**Đánh giá:** Các violation còn lại có trong phạm vi post-route optimization có thể recover không? Nếu WNS quá lớn → vấn đề cơ bản từ placement/CTS, không thể fix bằng routing.

### 21.2 DRC Check

**Ưu tiên tuyệt đối — Short và Open:**

Short và open **phải được giải quyết hoàn toàn trước khi tiếp tục**, vì:

- **Parasitic extraction** dựa trên netlist connectivity đúng. Nếu có short → extraction tính wrong RC → STA sai.
- **Open** nghĩa là net chưa được kết nối → STA không thể phân tích path đó chính xác.

**Spacing, width, enclosure violations:**

- Isolated (ít, rải rác) → có thể xử lý trong post-route optimization
- Widespread (nhiều, lan rộng) → dấu hiệu routing convergence problem → cần quay lại placement hoặc floorplan

### 21.3 Routing Quality Check

- **Overflow và hotspot count** phải gần bằng 0 ở giai đoạn này
- Review **antenna violations**
- Assess **SI/crosstalk exposure** trên critical net

---

## 22. Timing Path Groups

### 22.1 Bốn Nhóm Primary

|Group|Mô Tả|Launch → Capture|
|---|---|---|
|**in2reg**|Primary input đến register|Port input → Flip-flop D|
|**reg2reg**|Register đến register|Flip-flop Q → Flip-flop D|
|**in2out**|Primary input đến primary output|Port input → Port output|
|**reg2gout**|Register đến primary output|Flip-flop Q → Port output|

### 22.2 Các Nhóm Bổ Sung

|Group|Mô Tả|
|---|---|
|**in2icg**|Primary input đến ICG (Integrated Clock Gate) cell|
|**reg2icg**|Register đến ICG cell|
|**reg2mem**|Register đến memory (SRAM, ROM)|

### 22.3 Tại Sao Path Grouping Quan Trọng?

**Phân tích riêng lẻ:** Mỗi group có đặc điểm timing khác nhau. in2reg có ràng buộc từ input delay. reg2reg là nhóm lớn nhất. in2out có thể không cần clock.

**Targeted optimization:** Tool tối ưu hóa theo từng group — tránh "dành quá nhiều resource cho một group mà bỏ qua group khác".

**Prioritization:** Tool biết group nào quan trọng nhất để ưu tiên giải quyết.

**Debugging dễ hơn:** Khi có violation, ngay lập tức biết thuộc group nào → narrow down root cause.

---

## 23. Kiểm Tra Connectivity Của Routed Net

### 23.1 Các Loại Connectivity Issue

**Open net:**

- Wire bị đứt hoặc không hoàn chỉnh
- Hai điểm cần kết nối không được kết nối
- Nguyên nhân: Router không tìm được đường (severe congestion), hoặc pin bị block

**Shorted net:**

- Hai net khác nhau vô tình chạm nhau
- Tạo ra kết nối không mong muốn → functional failure
- Nguyên nhân: Spacing violation không được fix, rip-up không thành công

**Unconnected pin:**

- Pin của cell, macro, hoặc port không được router tiếp cận
- Subset của open net — pin cụ thể không có wire kết nối đến

**Incomplete routing:**

- Net chỉ được route một phần, một số branch còn thiếu

**Via/layer-connection problem:**

- Tín hiệu không kết nối đúng từ layer này sang layer khác
- Ví dụ: Via bị đặt sai layer, stack via không liên tục

---

## 24. Design Rule Checking (DRC) — Nguyên Lý Và Các Loại Vi Phạm

### 24.1 Tại Sao DRC Tồn Tại?

Layout "đẹp trên màn hình" không đồng nghĩa với "có thể sản xuất đáng tin cậy". Có nhiều hiện tượng vật lý trong quá trình fab gây sai lệch:

**Lithography limits (Giới hạn quang khắc):**

- Bước sóng ánh sáng có giới hạn độ phân giải vật lý
- Line quá mỏng hoặc space quá nhỏ sẽ không in được chính xác
- Kết quả: wire có thể bị đứt (open) hoặc dính vào wire kề (short)
- **Rule sinh ra**: Minimum width và minimum spacing

**Layer misalignment (Lệch lớp):**

- Trong lithography, mỗi mask layer có thể bị shift vài nm so với layer khác (overlay error)
- Via phải có đủ enclosure metal để vẫn kết nối đúng dù có overlay error
- **Rule sinh ra**: Enclosure rule, extension rule

**Etch và deposition variation:**

- Etching rate không đều → wire có thể dày hoặc mỏng hơn dự định
- **Rule sinh ra**: Width tolerance, spacing tolerance

**Particle contamination:**

- Hạt bụi nhỏ có thể phá wire mỏng hoặc nối wire gần nhau
- **Rule sinh ra**: Larger spacing để tăng tolerance với contamination

**Electrical reliability:**

- Width ảnh hưởng trực tiếp đến resistance và current capacity (EM limit)
- Spacing ảnh hưởng đến leakage, breakdown voltage
- Via và contact rules giảm failure risk

### 24.2 Các Loại DRC Violation

**Minimum spacing violation:**

- Hai wire/shape quá gần nhau → nguy cơ short trong fabrication
- Quan hệ: $S_{actual} < S_{minimum}$

**Minimum width violation:**

- Wire quá hẹp → khó fabricate, unreliable về điện
- Quan hệ: $W_{actual} < W_{minimum}$

**Via rule violation — 3 loại:**

1. **Enclosure violation**: Metal không bọc đủ rộng quanh via cut
2. **Via spacing violation**: Hai via quá gần nhau
3. **Cut-spacing violation**: Trong multi-cut via, khoảng cách giữa các cut quá nhỏ

**Minimum area violation:**

- Metal shape quá nhỏ về diện tích → không fabricate được đáng tin cậy
- Đặc biệt quan trọng ở các "island" metal nhỏ sau DRC fix

**Routing rule violation:**

- Wire off-track (không nằm trên routing track hợp lệ)
- Wire trên layer không được phép
- Notch violation: Hình dạng U-shape với notch quá nhỏ

### 24.3 Three Basic DRC Checks

Ba check cơ bản nhất trong mọi technology:

1. **Width**: Kiểm tra chiều rộng tối thiểu của wire
2. **Spacing**: Kiểm tra khoảng cách tối thiểu giữa hai shape
3. **Enclosure**: Kiểm tra via được bọc đủ bởi metal trên cả hai phía

---

## 25. Pin Access — Vấn Đề Tiếp Cận Pin

### 25.1 Định Nghĩa

Pin access problem xảy ra khi router không thể tiếp cận pin của cell một cách hợp lệ (không vi phạm DRC) bằng wire routing thông thường.

### 25.2 Nguyên Nhân

**Hình học cell placement:**

- Hai cell đặt cạnh nhau → wire của cell này chặn pin của cell kia
- Khoảng cách giữa hai cell (placement gap) ảnh hưởng trực tiếp đến pin accessibility

**Illustration từ slide (Figure 2):**

- **(a)**: Hai cell đặt liền kề nhau — wire M2 (horizontal) chạy qua, blocked pin (đánh dấu hatch) trên M1 không thể được tiếp cận từ hướng thông thường
- **(b)**: Hai cell đặt với gap 3 placement pitches — M3 wire (vertical, màu xanh lá) có thể đi xuống qua gap để reach pin
- **(c)**: Pin access failure trong detailed routing — pin blocked, không tìm được đường hợp lệ
- **(d)**: Pin access success — cell A được flip, pin được tiếp cận thành công qua path hợp lệ

### 25.3 Các Giải Pháp

**Detour hoặc via stack:**

- Wire phải đi vòng để tiếp cận pin từ hướng khác
- Dùng via stack (kết nối qua nhiều layer) để reach pin từ layer cao hơn

**Cell flipping:**

- Flip cell theo trục ngang → pin di chuyển sang vị trí khác → có thể accessible hơn
- Trade-off: Cell flip có thể ảnh hưởng placement legality

**Pin swapping:**

- Với cell có nhiều pin tương đương (ví dụ: NAND2 có thể swap hai input)
- Swap pin để pin dễ tiếp cận hơn được dùng

**DRC cleanup và incremental legalization:**

- Fix DRC violation phát sinh sau khi fix pin access

**ECO nếu vẫn thất bại:**

- Persistent pin access failure → Engineering Change Order → thay đổi placement hoặc netlist

---

## 26. Kiểm Tra Timing Sau Routing

### 26.1 Tại Sao Phải Re-check Timing Sau Routing?

|Stage|Trạng Thái Clock Net|Trạng Thái Signal Net|RC Source|
|---|---|---|---|
|CTS Optimization|Đã route và optimize|Chưa route (hoặc partially)|Signal RC = estimated|
|After route_design|Đã route|Đã route hoàn chỉnh|Signal RC = extracted (thực)|

Sau CTS optimization, timing analysis của signal net dùng **ước lượng** (không chính xác). Sau routing, dùng RC thực → kết quả timing mới là **ground truth** cho implementation flow.

### 26.2 Các Bước Kiểm Tra Timing Sau Routing

1. **RC extraction** (`extract_rc`): Tính parasitic R và C thực từ wire và via shapes
2. **Timing analysis** (`time_design -post_route`): Chạy STA với RC thực để kiểm tra setup, hold, skew, transition, capacitance, SI/noise impact
3. **Report QoR** (`report_qor`): Summary checkpoint cho WNS, TNS, violating path count, overall timing quality

### 26.3 Phân Biệt time_design và report_timing

**`time_design` — Stage-level timing health check:**

- Câu hỏi trả lời: _"Toàn bộ thiết kế đang ở mức nào về timing?"_
- Output: WNS, TNS, số endpoint vi phạm, setup/hold status, phân tích theo path group, tổng quan timing closure
- Khi dùng: Sau mỗi optimization stage để đánh giá tiến độ

**`report_timing` — Path-level debug:**

- Câu hỏi trả lời: _"Path nào đang fail và tại sao?"_
- Output chi tiết: Launch clock path, data path, capture clock path, cell delay từng gate, net delay từng wire, arrival time, required time, slack, slew/load
- Khi dùng: Khi muốn phân tích nguyên nhân cụ thể của một vi phạm

---

## 27. Tổng Quan Timing Path Groups — Bổ Sung

### 27.1 Cấu Trúc Mỗi Timing Path

Mỗi timing path gồm 3 phần:

1. **Launch clock path**: Từ clock source đến clock pin của launch flip-flop
2. **Data path**: Từ Q pin của launch flip-flop qua combinational logic đến D pin của capture flip-flop
3. **Capture clock path**: Từ clock source đến clock pin của capture flip-flop

**Setup check:** $$T_{launch_clk} + T_{data} \leq T_{capture_clk} + T_{period} - T_{setup}$$

**Hold check:** $$T_{launch_clk} + T_{data} \geq T_{capture_clk} + T_{hold}$$

### 27.2 ICG Path Groups

ICG (Integrated Clock Gate) là flip-flop dạng đặc biệt dùng để gating clock:

- **in2icg** và **reg2icg**: Path đến enable input của ICG — quan trọng vì nếu enable đến trễ → clock bị cắt sai → functional error
- ICG path thường có constraint đặc biệt về setup margin

---

## 28. Liên Kết Giữa Các Khái Niệm

### 28.1 Chuỗi Nhân Quả Từ Metal Choice Đến Timing

```
Metal width ↑ → R↓ → RC delay↓ → Slack↑ (tốt hơn)
                    → C tăng chút (do area) → trade-off nhỏ

Metal spacing ↑ → Cm (coupling) ↓ → Crosstalk↓ → SI↑
                → Routing resource↓ → Congestion↑ → trade-off

Via count ↓ → R↓ → timing↑ → EM risk↑ (nếu current lớn)
Via count ↑ → Area↑ → Congestion↑ → DRC phức tạp hơn
```

### 28.2 Technology File → Routing → Sign-off

```
LEF (technology) → PnR routing rules → Routing result
ITF (technology) → RC extraction → Actual parasitics → STA → Timing sign-off
DRC rules → check_drc → Physical sign-off
LVS rules → check_lvsv → Electrical connectivity sign-off
```

### 28.3 Congestion → DRC → Timing Relationship

- Congestion cao → router phải route detour dài → wire length tăng → RC tăng → timing xấu
- Congestion cao → router "ép" wire vào nhỏ hơn → spacing giảm → DRC violation + crosstalk tăng
- DRC violation (short) → extraction sai → STA sai → timing report không tin cậy

### 28.4 Tầm Quan Trọng Của Thứ Tự Ưu Tiên Fix

1. **Short/Open** phải fix đầu tiên: Nếu còn short/open → extraction sai → tất cả analysis sau đó không đáng tin
2. **Spacing/Width DRC** fix tiếp: Đảm bảo layout manufacturability
3. **Timing violation** fix sau: Chỉ có nghĩa khi DRC đã clean

---

## 29. Tóm Tắt Kiến Thức Cốt Lõi

|Khái Niệm|Vai Trò Trong Routing|
|---|---|
|Technology file (LEF/ITF)|Định nghĩa rules và RC model cho routing|
|Metal stack|Kiến trúc phân lớp với từng layer có mục đích riêng|
|Routing track & pitch|Grid cố định mà router đặt wire|
|Preferred direction|Giảm coupling, tăng routing efficiency|
|Via types|Trade-off giữa RC, reliability, area, DRC|
|Global routing|Lập kế hoạch routing ở mức GCELL|
|Detailed routing|Tạo wire và via thực tế, fix DRC|
|Post-route optimization|Refine timing/SI/EM dùng actual RC|
|DRC|Đảm bảo layout có thể sản xuất đáng tin cậy|
|Crosstalk|Coupling điện giữa wire kề nhau ảnh hưởng timing và function|
|NDR|Non-default rule — wire rộng hơn/spacing lớn hơn cho critical net|
|Routing blockage|Kiểm soát vùng router được phép đi qua|
|Routing constraint|Hướng dẫn cụ thể cho router về từng net/class/region|
|Pin access|Khả năng router tiếp cận pin hợp lệ không vi phạm DRC|
|RC extraction|Tính parasitics thực từ routed wire để STA chính xác|