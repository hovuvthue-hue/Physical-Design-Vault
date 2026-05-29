# 1. Design Import & Initialization (Nhập Dữ liệu & Khởi tạo Thiết kế) [Trang 2 - 4]

Dựa trên slide (trang 2 và 3), Design Import là pha đầu tiên trong luồng Physical Design (PD Flow). Để bắt đầu quá trình Floorplan, công cụ EDA cần được nạp toàn bộ Database (DB) từ khâu Logic Synthesis và các thông số vật lý của công nghệ.

**1. Khai báo các tệp Input Data cơ sở:** Kỹ sư sử dụng các lệnh TCL để trỏ công cụ tới các tệp dữ liệu cần thiết:

- `read_netlist chip.v` : Nạp Gate-level Netlist (mã nguồn RTL đã được tổng hợp thành các Standard Cells và Macros).
    
- `read_mmmc chip.mmmc` : Nạp tệp cấu hình MMMC (Multi-Mode Multi-Corner). Tệp này sẽ trỏ tới các thư viện `.lib` (Timing/Power views) và các tệp SDC (Timing Constraints).
    
- `read_physical cells.lef` : Nạp thư viện LEF (Library Exchange Format). Bao gồm Technology LEF (chứa Design Rules của các lớp Metal) và Cell/Macro LEF (chứa kích thước Bounding Box và tọa độ Pins).

**2. Khai báo Power / Ground Nets:** Trước khi dựng hình vi mạch, kỹ sư bắt buộc phải khai báo (explicitly define) tên của các lưới nguồn để công cụ nhận diện đúng cấu trúc Power Domain:

- `set_db init_power_nets VDD` : Khai báo Power Net (lưới nguồn dương).
    
- `set_db init_ground_nets VSS` : Khai báo Ground Net (lưới đất).

**3. Khởi tạo Thiết kế (Init Design):**

- `init_design` : Đây là lệnh thực thi cốt lõi. Công cụ sẽ compile toàn bộ Netlist, MMMC, LEF và thiết lập Power/Ground ở trên để dựng lên Database nội bộ trong bộ nhớ RAM, sẵn sàng cho khâu Floorplan.

**4. Post-Import Sanity Checks (Đánh giá sơ bộ sau khi Import):** Ngay khi `init_design` chạy xong, kỹ sư PD tuyệt đối không chuyển sang vẽ Floorplan ngay. Phải thực hiện khâu "Sanity Check" để đảm bảo SDC và Timing Libraries được nạp đúng cách, không có rác (Garbage In):

- `check_timing` : Rà soát toàn bộ thiết kế để tìm các lỗi thiếu sót trong SDC (ví dụ: Unconstrained Endpoints, Missing Clocks, vòng lặp Combinational Loops).
    
- `report_analysis_coverage` : Kiểm tra tỷ lệ bao phủ của các Timing Checks.
    
- `report_timing` và `report_slack_histogram` : Trích xuất báo cáo WNS/TNS và biểu đồ phân bổ Slack (Slack Histogram) sơ bộ (Zero-wire-load) để đánh giá Timing Feasibility trước khi bắt đầu Floorplan.

---
**Tiểu kết Phần 1:** Giai đoạn Design Import yêu cầu sự chính xác tuyệt đối về đường dẫn (paths) của các file đầu vào. Việc bỏ qua các lệnh `check_timing` ở bước này sẽ dẫn đến những lỗi ECO (Engineering Change Order) cực kỳ tốn thời gian ở giai đoạn Routing cuối cùng.

# 2. Floorplanning Overview & Objectives (Tổng quan và Mục tiêu của Floorplan) [Trang 5 - 10]

Đây là phần định hình tư duy trước khi bạn gõ bất kỳ câu lệnh thiết lập tọa độ nào. Floorplan không đơn thuần là việc "xếp hình" cho vừa vặn, mà là một bài toán tối ưu hóa đa biến, quyết định trực tiếp đến sự thành bại của các khâu Placement và Routing phía sau.

### 2.1. Bản chất của Floorplan

Floorplanning là quá trình ánh xạ (mapping) trực tiếp từ cấu trúc logic (Gate-level Netlist) sang biểu diễn vật lý (Physical footprint). Nó định hình không gian cơ sở để chuyển đổi các đoạn code HDL trừu tượng thành một hệ thống thực thể silicon có tọa độ rõ ràng.

### 2.2. Các Nhiệm vụ Chính (Primary Goals)

Trong khâu này, kỹ sư PD phải ra quyết định và thiết lập tọa độ cho các thành phần vật lý tĩnh (static components) sau:

- **Die & Core:** Xác định kích thước tổng thể (Size) và tỷ lệ khung hình (Aspect Ratio) của toàn bộ vi mạch.
    
- **I/O Pads & Ports:** Quyết định vị trí của các khối đệm giao tiếp ngoại vi (I/O Pads đối với thiết kế full-chip) hoặc các chân tín hiệu (Pins/Ports đối với thiết kế block-level).
    
- **Hard Macros:** Bố trí tọa độ cho các khối IP đúc sẵn có diện tích lớn (như SRAM, ROM, PLL, ADC/DAC).
    
- **Power / Ground Network:** Quy hoạch kiến trúc mạng lưới phân phối nguồn điện (Power Grid) để cấp $V_{DD}$ và $V_{SS}$ đồng đều cho toàn bộ vi mạch.

![[Pasted image 20260508155735.png]]
### 2.3. Ba Mục tiêu Tối ưu hóa (Key Objectives)

Kỹ sư đánh giá chất lượng của một bản Floorplan dựa trên 3 tiêu chí sinh tử sau:

1. **Minimize Area (Tối thiểu hóa diện tích):**
    
    Mục tiêu thương mại cốt lõi là giảm kích thước Die (Die size) càng nhiều càng tốt. Diện tích càng nhỏ, số lượng chip cắt ra từ một tấm Wafer càng nhiều, dẫn đến chi phí sản xuất (Silicon cost) giảm đi. Tuy nhiên, việc ép diện tích quá mức sẽ xung đột trực tiếp với 2 mục tiêu bên dưới.
    
2. **Timing Closure (Đảm bảo hiệu suất thời gian):**
    
    Vị trí đặt các Macros quyết định chiều dài của các tuyến đường dẫn (Nets). Kỹ sư phải bố trí các khối có tần suất giao tiếp cao (Dataflow mạnh) nằm gần nhau để giảm thiểu **Net Delay**. Nếu đặt sai, tín hiệu sẽ phải chạy xuyên qua toàn bộ Die, gây ra các vi phạm Setup Time không thể sửa chữa (Fix) ở các khâu sau.
    
3. **Minimize Routing Congestion (Giảm thiểu tắc nghẽn định tuyến):**
    
    Bản Floorplan phải chừa lại đủ không gian trống (White space) và các luồng đi dây (Routing Channels) hợp lý. Nếu các khối bị nhồi nhét quá chật chội, công cụ Routing sẽ cạn kiệt số lượng **Track** trên các lớp Metal, dẫn đến hiện tượng Congestion nghiêm trọng, đứt dây (Unrouted nets) hoặc vi phạm quy tắc thiết kế vật lý (**DRC**).

---
**Tiểu kết Phần 2:**
Một bản Floorplan xuất sắc là một điểm cân bằng hoàn hảo (Trade-off) giữa Diện tích nhỏ nhất, Timing tốt nhất và Congestion thấp nhất.

# 3. Spatial Control Parameters (Kiểm soát không gian vật lý: Aspect Ratio & Core Utilization) [Trang 11 - 16]

Trong môi trường thiết kế vật lý, sau khi đã xác định được mục tiêu, kỹ sư phải phiên dịch chúng thành các thông số toán học để định hình cấu trúc không gian cho công cụ **EDA**. Lệnh khởi tạo mặt bằng (ví dụ: lệnh `floorPlan` trong Innovus) dựa hoàn toàn vào hai thông số nền tảng sau:

![[Pasted image 20260508162938.png]]

### Tổng quan

**3.1. Aspect Ratio (Tỷ lệ khung hình) [Trang 11]**

**Aspect Ratio (AR)** quy định hình dáng hình học của **Core Area**.

- **Công thức toán học:** $AR = \frac{\text{Height}}{\text{Width}}$
    
- **Ứng dụng thực tiễn:**
    
    - Nếu $AR = 1.0$: **Core** có dạng hình vuông. Đây luôn là cấu trúc lý tưởng và được ưu tiên nhất cho các thuật toán **Placement** và **Routing**, vì quãng đường trung bình mà tín hiệu phải di chuyển theo trục X và trục Y là cân bằng nhau, giúp giảm thiểu **Net Delay**.
        
    - Nếu $AR > 1.0$ (chữ nhật đứng) hoặc $AR < 1.0$ (chữ nhật ngang): Thường bắt buộc phải thiết lập khi vi mạch bị giới hạn bởi cấu trúc cơ học của **Package** hoặc do yêu cầu phân bổ số lượng **I/O Pads** không đồng đều ở 4 cạnh.

**3.2. Core Utilization (Mật độ sử dụng lõi) [Trang 12 - 13]**

Nếu **Aspect Ratio** quyết định "hình dáng", thì **Core Utilization** quyết định "độ chật chội" của thiết kế. Nó là tỷ lệ phần trăm diện tích lõi bị chiếm dụng bởi các linh kiện vật lý.

- **Công thức [Trang 13]:**$$Utilization = \frac{\text{Total Area of Standard Cells} + \text{Total Area of Macros}}{\text{Total Core Area}}$$
- **Khái niệm White space:** Phần diện tích không bị chiếm dụng (bằng $100\% - Utilization$) được gọi là khoảng trống (**White space**). Khoảng trống này không hề vô dụng, nó là tài nguyên sống còn để cấp phát cho lưới định tuyến (**Routing Tracks**).
    

**3.3. Chiến lược đánh đổi (Trade-offs) và Hệ quả [Trang 15 - 16]**

Kỹ sư **Physical Design** phải quyết định mức **Utilization** một cách vô cùng cẩn trọng:

- **Mật độ quá cao (High Utilization - ví dụ > 85%):**
    
    - _Lợi ích:_ Kích thước **Die Area** nhỏ, tối đa hóa lợi nhuận trên mỗi tấm Wafer.
        
    - _Hậu quả:_ Sự cạn kiệt **White space** dẫn đến thiếu hụt nghiêm trọng các **Routing Tracks**. Hậu quả tất yếu là công cụ sẽ báo lỗi **Routing Congestion** (tắc nghẽn đi dây) và sinh ra hàng loạt vi phạm **DRC**. Ngoài ra, ở giai đoạn sau, khi cần sửa lỗi **Timing** (thông qua quá trình **ECO**), phần mềm sẽ không tìm được khoảng trống nào để chèn thêm các **Buffers** hay **Inverters**.
        
- **Mật độ quá thấp (Low Utilization - ví dụ < 60%):**
    
    - _Lợi ích:_ Rất dồi dào tài nguyên **Routing**, dễ dàng đạt được **Timing Closure**.
        
    - _Hậu quả:_ Lãng phí diện tích Silicon vô ích (vượt ngân sách dự án), đồng thời khoảng cách phân tán giữa các **Standard Cells** quá xa có thể làm tăng điện dung ký sinh trên đường dây, gây suy hao hiệu suất.
        
- _Khuyến nghị tiêu chuẩn:_ Trong các dự án bắt đầu luồng thiết kế, kỹ sư thường thiết lập **Core Utilization** ở mức an toàn là **70% (0.7)**.

### 3.1. Cấu trúc Hình học Tổng thể (Geometric Relationships - Slide 11)

Bản vẽ tại Slide 11 phân định rõ ranh giới giới hạn của các vùng không gian:

- **Core-to-IO Spacing (Khoảng đệm ngoại vi):** Khoảng cách từ viền ngoài của khối logic đến vành đai **I/O Pads**. Khoảng trống này phải đủ rộng để chứa cấu trúc lưới nguồn **Power Rings** và mạng lưới định tuyến I/O.
    
- **Core Area (Vùng lõi):** Là tổng diện tích khả dụng nằm bên trong ranh giới viền lõi.
    
- **Standard Cell Area & Macros:** Hai thành phần này cấu thành nên phần lõi logic. Các **Macros** (như SRAM, IP) là các khối vật lý đóng băng (hard IP), chiếm các khoảng diện tích hình chữ nhật khổng lồ, phần diện tích còn lại mới được cấp phát cho **Standard Cells**.

### 3.2. Giải phẫu chi tiết Không gian Lõi (Core Area Anatomy - Slide 12)

Đây là phần giải phẫu quan trọng nhất. Diện tích lõi thực tế không bao giờ được sử dụng 100% cho linh kiện logic. Sơ đồ tại Slide 12 phân rã **Core Area** thành các vùng nhỏ hơn:

![[Pasted image 20260508164817.png|715]]

**A. Unusable Area (Vùng không khả dụng):** Diện tích bị khóa hoàn toàn đối với **Standard Cells**, được tạo ra bởi hai nguyên nhân chính:

1. **Macro Padding (Vành đai bảo vệ Macro):** Khoảng không gian (thường là **Halos**) bao quanh các khối **Macros** để chừa đường cho các lớp kim loại cắm vào **Pins** của Macro.
    
2. **Blockage Area (Vùng cấm Placement):** Các khu vực do kỹ sư thiết lập (như Hard/Soft Placement Blockages) để ép công cụ không được rải **Standard Cells** vào, nhằm mục đích quản lý tắc nghẽn (Congestion).

**B. Placeable Standard Cell Area (Vùng khả dụng cho Standard Cell):** Phần diện tích còn lại sau khi đã trừ đi diện tích của **Macros** và **Unusable Area**. Vùng này lại tiếp tục bị chia nhỏ thành 3 phần:

1. **Standard Cells (Đã sử dụng):** Diện tích thực tế bị chiếm dụng bởi các cổng logic từ **Netlist** ban đầu.
    
2. **Additional Standard Cells Area (Diện tích dự phòng ECO):** Không gian trống bắt buộc phải được quy hoạch sẵn để công cụ chèn thêm các **Buffers**, **Inverters** ở bước **Clock Tree Synthesis (CTS)** và quá trình tối ưu hóa **Timing** (Hold/Setup fixing).
    
3. **Empty Area between SCs (Khoảng trống giữa các SCs):** Đây là các khe hở rải rác giữa các cell. Nó cung cấp không gian để rớt các Vias (lỗ xuyên) cho công đoạn **Routing** và chèn các **Tap Cells**.

### 3.3. Tác động của Utilization lên Hiệu suất (Routability & SI Impacts)

Sự tương tác giữa **Design Rules** (Luật thiết kế từ nhà máy) và mức **Utilization** (Mật độ sử dụng) sẽ sinh ra hệ quả vật lý trực tiếp:

- **Design Rules:** Yêu cầu khoảng cách tối thiểu giữa các dây dẫn kim loại (Min Spacing) và bề rộng dây tối thiểu (Min Width) khiến cho việc định tuyến các **Nets** tốn rất nhiều diện tích bề mặt.
    
- **Khi Utilization quá cao:** **Empty Area between SCs** bị nén lại gần bằng không. Các đường dây dẫn bị dồn ép vào một hành lang cực kỳ chật hẹp.
    
    - **Routability Impact (Tác động định tuyến):** Thiếu **Track** để đi dây, công cụ bắt buộc phải đi vòng (detour) làm tăng **Net Delay**, hoặc báo lỗi ngắn mạch (Shorts).
        
    - **SI Impacts (Tác động Toàn vẹn Tín hiệu - Signal Integrity):** Các dây dẫn chạy song song sát nhau ở mật độ cao sẽ sinh ra điện dung ký sinh xuyên âm (Crosstalk/Coupling Capacitance). Nhiễu xuyên âm này có thể làm thay đổi hoàn toàn **Transition/Slew** của tín hiệu gốc, dẫn đến sai lệch dữ liệu hoặc vi phạm **Timing** trầm trọng ở giai đoạn hậu kỳ (Post-route).

Ngược lại, nếu thiết lập **Utilization** quá thấp để tối đa hóa **Empty Area**, mặc dù vi mạch sẽ có **Routability** cực kỳ lý tưởng và miễn nhiễm với **SI Impacts**, nhưng thiết kế sẽ vi phạm các yêu cầu về chi phí thương mại (do lãng phí silicon) và tiêu thụ điện năng (do độ dài dây trung bình tăng lên).

# Trang 9, 10 - PnR Boundary

