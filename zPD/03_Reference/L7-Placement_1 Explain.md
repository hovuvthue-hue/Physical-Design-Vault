
# 1. Introduction

## Trang 5, 6, 7, 8

Tác giả muốn đi thẳng vào vấn đề bằng cách giới thiệu bước hành động chính (Placement - ta đang làm gì?), sau đó giới thiệu đối tượng bị tác động (Standard Cell - ta đặt cái gì?), và cuối cùng mới đi sâu mổ xẻ nền tảng vật lý bên dưới (Site và Row - không gian vật lý chứa nó là gì?)

#### A. Site (Đơn vị lưới cơ sở - Trang 7) ^site1

Mọi thứ trong không gian vật lý đều phải dựa trên một hệ quy chiếu:

- **Site** định nghĩa một "minimum grid unit" (đơn vị lưới nhỏ nhất bao gồm _width_ và _height_) dùng để đặt một **Standard Cell**.
    
- Các **Standard Cell** bắt buộc phải được "align" (căn chỉnh) theo **site grid** này bên trong các **placement Row**.
    
- Tùy thuộc vào kích thước, các cell lớn hơn có thể nằm vắt ngang (span) qua nhiều **Site** theo chiều _width_ và/hoặc _height_.

#### B. Row (Hàng chứa Cell - Trang 8) ^row1

Từ các **Site** riêng lẻ ghép lại, ta có cấu trúc hàng:

- Một **Row** được tạo thành từ một chuỗi các **Site** nằm liền kề nhau (adjacent).
    
- **Row** được định nghĩa bởi _technology library_ và đóng vai trò là "placement structure" (cấu trúc sắp xếp) cho các **Standard Cell**. Nó đại diện cho "region" (vùng) hợp lệ để đặt các cell này.
    
- Về kích thước, **Row** có "fixed height" (chiều cao cố định) để nhất quán với các **Standard Cell** mà nó chứa. Tuy nhiên, trong các thiết kế tiên tiến (advanced/lower geometry), có thể tồn tại nhiều "row height" khác nhau.
    
- Các **Row** có thể có nhiều "orientation" (hướng) khác nhau như _R0_, _MX_, hoặc cấu trúc lật/tiếp giáp (_flipped/abut_). Việc thiết lập _abut_ cho phép hai **Row** nằm cạnh nhau có thể kết nối chung nguồn _VDD_ hoặc chung đường đất _VSS_.

#### C. Standard Cell (Khối thiết kế cơ bản - Trang 6) ^StandardCell1

Khi đã có nền tảng không gian (**Site/Row**), ta sẽ xem xét đối tượng được đặt vào:

- **Standard Cell** là một "logic cell" đã được pre-characterized (đặc tả sẵn) và định nghĩa rõ ràng, đóng vai trò là "basic building block" (khối xây dựng cơ bản) trong thiết kế.
    
- Về mặt vật lý, chúng được thiết kế với "uniform height" (chiều cao đồng nhất) để có thể align vừa vặn vào các **Standard Cell Row**.
    
- Mặc dù có "fixed height", nhưng _width_ của chúng lại biến đổi (variable width) phụ thuộc vào _functionality_ (chức năng của cell đó).
    
- **Standard Cell** cũng có thể có sẵn dưới dạng nhiều biến thể chiều cao khác nhau, ví dụ như _single-height_ (chiều cao đơn) hoặc _double-height_ (chiều cao kép).

#### D. Placement (Quá trình sắp xếp - Trang 5)

Cuối cùng, đây là bức tranh toàn cảnh kết hợp tất cả các yếu tố trên:

- **Placement** là giai đoạn mà các **Standard Cell** từ _netlist_ được chỉ định các "physical location" (vị trí vật lý) cụ thể vào bên trong "core area" của chip (đặt vào các **Site** và **Row**).
    
- Ngoài việc định vị trí cho các cell, bản thân _netlist_ cũng được optimize (tối ưu hóa) ngay trong giai đoạn **Placement** này nhằm đạt được mục tiêu PPA (Power, Performance, Area) tốt hơn.

## Trang 9, 10: Standard Cell Placement

### 1. Standard Cell Placement Area & Placement Row (Không gian sắp xếp)

Trước khi đặt bất kỳ thành phần nào, ta cần xác định không gian hợp lệ của chúng trên chip:

- **Core Area (Vùng lõi):** Quá trình **Placement** diễn ra trong giới hạn nghiêm ngặt. **Tool** thiết kế sẽ **không** đặt bất kỳ **Standard Cell** nào ra bên ngoài ranh giới của **Core Area** (thường được đánh dấu bằng vùng màu đỏ trong layout).
    
- **Placement Row (Hàng sắp xếp):** Không gian hợp lệ bên trong lõi được chia thành các **Placement Row**. Đây là các dải dóng ngang (horizontal bands) được phân bổ đều nhau, đóng vai trò là "vùng hợp pháp" (legal regions) để chứa **Standard Cell**.
    
- Cấu trúc của **Placement Row** được xây dựng từ các **Site** hợp lệ. Kích thước và sự căn chỉnh của các **Site** này phải khớp tuyệt đối với thư viện **Standard Cell**. Các **Row** được căn chỉnh theo **legal site grid** (lưới site) và bị chi phối bởi **[[3.2. LEF - Library Exchange Format#^track1|track]] height** (chiều cao track, ví dụ: thư viện loại 7-track, 9-track).
### 2. Standard Cell Constraints (Ràng buộc vật lý của Cell)

Khi đặt vào các **Row**, kích thước và vị trí của các **Standard Cell** phải tuân thủ hệ thống lưới (grid):

- **Cell Height (Chiều cao):** Mọi cell phải có **Uniform Height** (chiều cao đồng nhất). Chiều cao này bắt buộc phải là bội số của **horizontal routing grid** (lưới định tuyến ngang).
    
- **Cell Width (Chiều rộng):** Có thể linh hoạt, nhưng phải là bội số của **Site Width** (chiều rộng của Site) và đồng thời là bội số của **vertical routing grid** (lưới định tuyến dọc).
    
- **Cell Pins & Origin:** Điểm gốc của cell được xác định qua **Cell Origin**. Để tối ưu hóa quá trình đi dây, các **Pin** (chân tín hiệu) của cell nên nằm trúng vào vị trí giao điểm (intersection) của cả lưới ngang và lưới dọc.

### 3. Standard Cell Placement Orientation (Hướng đặt Cell)

Để đảm bảo khả năng ghép nối, các **Standard Cell** có thể được lật hoặc xoay theo 4 **Orientation** (hướng) hợp lệ:

- **R0:** Đây là hướng mặc định (Default orientation). Ở hướng này, cell đứng thẳng (Upright), không xoay (no rotation) và không bị lật gương (no mirror).
    
- **MY:** Cell bị lật gương qua trục Y (Mirror along the Y-axis). Tuỳ thuộc vào hệ quy chiếu, sự biến đổi này còn được mô tả là lật từ trên xuống dưới (Flip top to bottom).
    
- **MX:** Cell bị lật gương qua trục X (Mirror along the X-axis).
    
- **R180:** Cell được xoay 180 độ ($180^{\circ}$ rotation).

### 4. Row Orientation & Cell Abutment (Tối ưu ghép nối nguồn và giếng)

Tại sao ta lại cần thay đổi **Orientation** của các cell?

- Việc lật/xoay cell nhằm mục đích cốt lõi là đảm bảo sự căn chỉnh chính xác (proper alignment) với hệ thống **Power Rail** (đường ray nguồn) và cho phép các cell có thể tiếp giáp hợp lệ (valid cell abutment) với nhau.
    
- **Row Orientation:** Thay vì đặt tất cả các hàng cùng một hướng, hệ thống sẽ xếp xen kẽ (alternates) giữa hàng mang hướng **R0** và hàng mang hướng **MY** để cho phép lật cell (cell flipping) giữa các hàng liền kề.
    
- **Cell Abutment:** Kết quả của cấu trúc lật xen kẽ này là sự hình thành các **Abutted PG Rail** (đường ray nguồn/đất tiếp giáp nhau), cho phép hai hàng sát nhau dùng chung dây **VDD** hoặc chung dây **VSS**. Đồng thời, nó cũng tạo ra các **Abutted N-Well** (chia sẻ chung giếng bán dẫn N-Well) để tiết kiệm tối đa diện tích silicon.

## Trang 11, 12, 13: Placement (Before vs After), Placement Density, Placement Blockages (Hard, Partial, Soft)

### 1. Sự chuyển đổi vật lý của thiết kế (Placement: Before vs After - Trang 11)

Quá trình **Placement** đóng vai trò là bước chuyển giao, biến đổi một "logical netlist" (danh sách kết nối logic) thành một "physically realizable layout" (bố cục vật lý khả thi) để sẵn sàng cho các bước tiếp theo như CTS và Routing .

- **Before Placement (Trước khi sắp xếp):** Các **Standard Cell** chưa được chỉ định các vị trí vật lý cụ thể ("physical locations") . Thiết kế nhìn rất rời rạc (sparse), thiếu tổ chức và không có bất kỳ sự ánh xạ (mapping) rõ ràng nào vào các **placement rows** hay các **sites** .
    
- **After Placement (Sau khi sắp xếp):** Các **Standard Cell** lúc này đã được đặt gọn gàng vào trong các hàng đã định (defined rows) và được căn chỉnh (aligned) tuyệt đối vào các **sites** . Bố cục layout trở nên có cấu trúc rõ ràng và dày đặc (dense) hơn . Giai đoạn này cũng đi kèm việc tối ưu hóa sơ bộ để cải thiện wirelength (chiều dài dây), timing (thời gian) và tổng thể PPA .

### 2. Kiểm soát không gian tổng thể: Placement Density (Mật độ sắp xếp - Trang 12)

Khi đưa cell vào không gian thiết kế, công cụ phải kiểm soát **Placement Density** – tức là tỷ lệ (fraction) diện tích khả dụng đã bị chiếm dụng bởi các **Standard Cell** .

- **Cell Density:** Tỷ lệ giữa tổng diện tích của các **Standard Cell** so với tổng "placeable area" (diện tích có thể đặt cell) .
    
- **Core Density:** Tỷ lệ lớn hơn, tính tổng diện tích của cả **Standard Cell** và các khối **Macros** so với tổng diện tích có thể đặt .
    
- **Ràng buộc trong thực tế:** Kỹ sư thường giới hạn **Placement Density** ở mức dưới ~70% để đảm bảo tính "routability" (khả năng đi dây dễ dàng sau này) . Nếu để mật độ quá cao (higher density), thiết kế sẽ dễ gặp tình trạng gia tăng "congestion" (tắc nghẽn) và gây ra nhiều thách thức trong khâu routing .

### 3. Kiểm soát vùng chi tiết: Placement Blockages (Vật cản - Trang 13)

Ngoài mật độ chung, kỹ sư cần sử dụng các **Placement Blockages** để điều khiển sự phân bổ placement, giảm tắc nghẽn và bảo vệ các khu vực trọng yếu (critical regions) trên chip . Có 3 loại vật cản chính:

- **Hard Blockage:** Cấm tuyệt đối. Công cụ bị hạn chế hoàn toàn, không có bất kỳ **Standard Cell** nào được phép đặt vào khu vực này .
    
- **Partial Blockage:** Hạn chế một phần. Công cụ vẫn được phép đặt cell vào vùng này, nhưng bị giới hạn "cell density" (mật độ cell) ở một mức tỷ lệ phần trăm được chỉ định trước .
    
- **Soft Blockage:** Giới hạn đặc biệt. Vùng này cấm các "regular cells" (cell logic thông thường), nhưng lại cho phép tool chèn các "buffers" hoặc "inverters" (nhằm mục đích tối ưu hóa) lên đến một mức mật độ xác định .

### 4. Tổng hợp các Câu lệnh (Innovus Commands)

Từ nguyên lý trên, dưới đây là cách các ràng buộc vật lý được thể hiện qua câu lệnh, đi từ kiểm soát tổng thể đến chi tiết:

**Lệnh kiểm soát Density tổng thể (Trang 12):**

- Cài đặt giới hạn mật độ tối đa trên toàn cục thiết kế là 70%: `set_db place_global_max_density 0.7`

**Các lệnh tạo Blockage khoanh vùng (Trang 13):**

- **Tạo Hard Blockage** (cấm hoàn toàn) cho một vùng bị tắc nghẽn thông qua tọa độ: `create_place_blockage -area 516.33950 288.29300 532.47700 301.35650 -type hard`
    
- **Tạo Partial Blockage** (cho phép mật độ 50%) tại một vùng cụ thể: `create_place_blockage -area 475.61200 331.32600 516.59600 358.22150 -type partial -density 50`
    
- **Tạo Soft Blockage** (cho phép mật độ 25% chỉ dành cho buffer/inverter): `create_place_blockage -area 535.80700 341.57200 562.44650 368.72350 -type soft -density 25`

## Trang 14, 15, 16: Objectives, Key Inputs, Flow Overview of Placement

### 1. Dữ liệu Đầu vào (Key Inputs to Placement - Trang 15)

Trước khi công cụ (Tool) có thể bắt đầu quá trình Placement, nó đòi hỏi phải có đầy đủ các dữ liệu nền tảng thiết yếu sau:

- **Netlist:** Danh sách kết nối logic mô tả mạch.
    
- **Physical Libraries ([[3.2. LEF - Library Exchange Format#^LEF1|LEF]]):** Thư viện chứa các thông tin vật lý (kích thước, chân pin...) của các phần tử.
    
- **Floorplan (DEF hoặc Database):** Bản vẽ mặt bằng chip đã được định hình từ bước trước đó.
    
- **MCMM Setup (Multi-Corner Multi-Mode):** Cấu hình thiết lập để phân tích ở nhiều chế độ và góc độ hoạt động khác nhau, bao gồm:
    
    - **Timing Libraries (.lib):** Thư viện chứa các thông số về thời gian trễ của cell.
        
    - **Timing Constraints (SDC):** Các tệp ràng buộc thời gian (như chu kỳ xung nhịp, độ trễ ngoại vi).
        
    - **RC Corners:** Các mô hình tham số điện trở (R) và điện dung (C) ký sinh.
        
- **Power Constraints (UPF):** Các ràng buộc về định dạng năng lượng thống nhất, đặc biệt quan trọng đối với các thiết kế có sử dụng nhiều mức điện áp khác nhau (multi-voltage designs).

### 2. Mục tiêu của giai đoạn Placement (Objectives of Placement - Trang 14)

Sử dụng các đầu vào kể trên, quá trình Placement hướng tới 5 mục tiêu cốt lõi:

- **Achieve timing and power goals:** Đảm bảo các ràng buộc về thời gian (setup/hold timing) được thỏa mãn, đồng thời tối ưu hóa để mức tiêu thụ năng lượng (power) là thấp nhất.
    
- **Minimize congestion:** Phân tán các cell một cách thông minh để tránh tạo ra các nút thắt cổ chai về đi dây (routing bottlenecks), qua đó cải thiện khả năng đi dây tổng thể (overall routability).
    
- **Control placement density:** Duy trì và kiểm soát mật độ đặt cell ở mức độ tối ưu. Phải chừa lại không gian (space) cho các bước tối ưu hóa thời gian và hệ thống đi dây sau này.
    
- **Avoid design rule violations (DRVs):** Ngăn chặn các vi phạm quy tắc thiết kế bằng cách tôn trọng các ràng buộc về: khoảng cách cell (cell spacing), thời gian chuyển tiếp tín hiệu tối đa (max transition), mức điện dung tối đa (max capacitance) và mức tải tối đa (max fanout).
    
- **Enable a routable design:** Mục tiêu bao trùm là thiết lập một bố cục layout chuẩn mực, tạo điều kiện thuận lợi và đảm bảo sự thành công cho các bước tiếp theo như Tổng hợp cây đồng hồ (CTS) và Đi dây (Routing).

### 3. Lưu đồ thực hiện (Placement Flow Overview - Trang 16)

Để hiện thực hóa các mục tiêu trên, luồng công việc Placement diễn ra qua 3 phân đoạn chính với các vòng lặp tối ưu liên tục (Iterations):

- **Giai đoạn 1: Pre-Placement (Trước khi đặt cell)**
    
    - Bước này chủ yếu thực hiện các **Sanity checks** (Kiểm tra tính toàn vẹn) nhằm rà soát lại xem dữ liệu đầu vào và Floorplan đã chuẩn xác chưa trước khi tool can thiệp.
        
- **Giai đoạn 2: Placement (Quá trình đặt cell chính)**
    
    - **Global placement:** Đặt rải rác thô các cell lên bản đồ chip để tìm vị trí tối ưu ban đầu.
        
    - **Detailed placement:** Điều chỉnh vi chỉnh, đưa các cell vào nằm gọn và hợp pháp bên trong các lưới (sites) và hàng (rows).
        
    - **High fanout net synthesis (HFNS):** Tổng hợp và đệm các đường tín hiệu có quá nhiều điểm nhận (ví dụ như tín hiệu reset).
        
    - **Scan chain reordering:** Sắp xếp lại chuỗi scan (dùng cho việc test chip) dựa trên vị trí vật lý để tiết kiệm dây dẫn.
        
- **Giai đoạn 3: Post-Placement (Sau khi đặt cell)**
    
    - Tiến hành **DRV fixing** (sửa các lỗi vi phạm vật lý) và **PreCTS timing optimization** (tối ưu hóa thời gian trước khi vẽ cây đồng hồ).
        
    - Chạy các bước tối ưu hóa cho Năng lượng (Power) và Diện tích (Area).
        
    - Chèn các **Tie-cells** (cell ghim mức điện áp chuẩn) và thêm các **Spare cells** (cell dự phòng cho các chỉnh sửa phút chót ECO).
        
    - Tối ưu hóa các cụm Flip-flop đa bit (**MBFF optimization**).
        
    - Tiến hành phân tích tắc nghẽn (**Congestion analysis**) và áp dụng các hướng dẫn (guides) để giảm thiểu tắc nghẽn.

# Pre-Placement - Trang 18, 19

### 1. Mục tiêu của Pre-Placement Sanity Checks (Kiểm tra tính toàn vẹn)

Trước khi công cụ thực sự bắt đầu giai đoạn đặt cell (Placement), kỹ sư bắt buộc phải tiến hành bước **Sanity checks** (Kiểm tra tính toàn vẹn) . Mục đích của bước này là để rà soát mặt bằng thiết kế (Floorplan), đảm bảo mọi thành phần nền tảng đã sẵn sàng và chuẩn xác trước khi công cụ tự động can thiệp.

### 2. Các tiêu chí vật lý bắt buộc phải đạt (Nội dung Trang 18)

Để một thiết kế vượt qua vòng "Sanity checks", nó phải thỏa mãn 5 điều kiện sau:

- **I/O Ports & Pins (Cổng và Chân tín hiệu):** Tất cả phải được đặt đúng vị trí đã chỉ định và được cố định (properly fixed) để không bị xê dịch .
    
- **Standard Cell Rows (Hàng chứa Cell):** Toàn bộ các hàng (Row) phải được tạo sẵn và căn chỉnh chính xác (aligned correctly) để làm không gian chứa các cell logic .
    
- **Macros (Các khối chức năng lớn):** Phải được đặt đúng vị trí và khóa chặt (locked) lại .
    
- **Halos & Blockages (Vùng bảo vệ và Vùng cấm):** Các vùng bảo vệ xung quanh macro (macro halos) và các khu vực cấm đặt (placement blockages) phải được định nghĩa rõ ràng để ngăn tool đặt cell sai chỗ .
    
- **PG Nets (Lưới Nguồn và Đất):** Hệ thống dây nguồn (Power) và dây đất (Ground) phải được kết nối đầy đủ và tuyệt đối không bị hiện tượng ngắn mạch (free of shorts) .

### 3. Thực thi bằng Câu lệnh Innovus và Phân tích Log (Nội dung Trang 19)

Để kiểm chứng 5 tiêu chí trên, tài liệu cung cấp hai nhóm câu lệnh (Commands) chính trên môi trường Innovus:

**A. Lệnh kiểm tra Floorplan (Kiểm tra cấu trúc mặt bằng):**

- **Câu lệnh:** `check_floorplan > check_floorplan.rpt` .
    
- **Tác dụng:** Câu lệnh này rà soát toàn diện các thiết lập về macros, cells, rows, blockages... và xuất kết quả ra file báo cáo `check_floorplan.rpt`.
    
- **Phân tích Log:** Khi chạy lệnh, phần mềm sẽ hiển thị tiến trình rà soát như: kiểm tra lưới định tuyến (routing tracks), ranh giới lõi/chip (core/die box), kiểm tra hàng (Checking Row is on grid, Checking row out of die) và các vùng cấm (Checking placement blockage) .
    
- **Phát hiện lỗi:** Nếu thiết kế vi phạm tiêu chí ở Trang 18, log sẽ xuất hiện cảnh báo (WARNING). Ví dụ:
    
    - Lỗi `IMPFP-10013` cảnh báo thiếu vùng bảo vệ quanh macro (`Halo should be created around block...`) .
        
    - Lỗi một phần tử không được gắn đúng vào lưới hàng (`not snapped to row-site`) .
        
    - Lỗi ranh giới Die/Core không khớp lưới (cảnh báo `IMPFP-7235`, `IMPFP-7237`) .

**B. Lệnh kiểm tra Lưới nguồn/đất (Kiểm tra PG Shorts):**

- **Câu lệnh:** `check_drc -check_short_only` .
    
- **Tác dụng:** Lệnh này chuyên dùng để đáp ứng tiêu chí cuối cùng ở trang 18 – rà soát toàn bộ thiết kế để tìm các điểm bị ngắn mạch trên lưới điện.
    
- **Phân tích Log:** Phần mềm sẽ chia mặt bằng chip ra thành các khu vực nhỏ (Sub-Area) và chạy lệnh `VERIFY DRC` trên từng vùng . Nếu log trả về dòng chữ `complete 0 Viols`, điều đó chứng tỏ vùng đó hoàn toàn an toàn, không có lỗi ngắn mạch (0 Violations) .

**Kết luận về cách trình bày của tác giả:** Việc tác giả sắp xếp Trang 18 trước rồi đến Trang 19 là một trình tự rất logic theo nguyên tắc "What & How". Trang 18 đóng vai trò thiết lập tư duy, cho kỹ sư biết **Mục tiêu cần làm là gì (What to check)**. Ngay sau đó, Trang 19 trao cho kỹ sư **Công cụ thực thi (How to check)**. Các dòng log hiển thị ở trang 19 chính là minh chứng thực tế giúp kỹ sư tự đối chiếu lại xem các tiêu chuẩn khắt khe ở trang 18 đã thực sự được hệ thống thỏa mãn hay chưa.

# Placement
## Trang 22 - 26: Global Placement

### 1. Nền tảng không gian: Routing Resources & Constraints (Trang 23, 24)

Trước khi đặt cell, hệ thống phải xác định khung giới hạn vật lý phục vụ cho việc kết nối dây dẫn sau này.

- **Routing Metal Layers (Các lớp kim loại định tuyến - Trang 23):** Các thiết kế vi mạch hiện đại thường sử dụng từ 3 đến 19 lớp kim loại để đi dây. Các lớp kề nhau sẽ giao tiếp thông qua các lỗ đâm xuyên gọi là **Vias**. Điểm quan trọng nhất là quy tắc **Preferred Routing Directions** (Hướng ưu tiên): các lớp liền kề nhau sẽ được quy định chạy vuông góc, luân phiên giữa chiều ngang (Horizontal) và chiều dọc (Vertical). Ví dụ: Metal 1 chạy ngang, Metal 2 chạy dọc, Metal 3 chạy ngang. Cấu trúc phân lớp đan xen này giúp tăng hiệu suất đi dây, giảm tắc nghẽn (congestion) và đảm bảo khả năng sản xuất.
    
- **Routing Tracks & Pitch (Đường ray và Bước lưới - Trang 24):** Trên mỗi lớp kim loại, dây dẫn không được vẽ tùy tiện mà bắt buộc phải chạy dọc theo các đường ray định sẵn gọi là **Routing Tracks**. Các track này được bố trí ngang dọc tạo thành một mạng lưới có cấu trúc (structured grid), giúp công cụ bám theo và tuân thủ chặt chẽ các quy tắc thiết kế. Khoảng cách tiêu chuẩn giữa hai track kề nhau được gọi là **Pitch**.

### 2. Quá trình thực thi: Global Placement & Global Routing (Trang 22, 25)

Khi đã nắm rõ không gian vật lý, phần mềm tiến hành rải cell thô và ước lượng tài nguyên đường dây.

- **Global Placement (Sắp xếp toàn cục - Trang 22):** Đây là bước phân bổ thô ban đầu, nơi các **Standard Cells** được rải rác trên toàn bộ vùng lõi (core area). Mục tiêu cốt lõi của khâu này là tối ưu hóa tổng chiều dài dây (wirelength) và giảm tắc nghẽn để đạt được lợi ích PPA (Power, Performance, Area) sớm. Ở giai đoạn sơ khởi này, luật vị trí được nới lỏng: các cell chưa cần phải căn chỉnh khít vào các **Placement Rows**, và phần mềm cho phép các cell có thể chồng lấn lên nhau (overlaps are allowed). Bản nháp này sẽ được điều chỉnh hợp pháp hóa (legalization) ở các bước tiếp theo.
    
- **Global Routing / Early Routing Estimation (Dự đoán đi dây - Trang 25):** Để định hướng cho quá trình Global Placement ở trên, công cụ phải đánh giá xem việc đặt cell đó có gây kẹt dây hay không. Nó chia mặt bằng chip thành một lưới thô bao gồm các ô **GCELLs**. Công cụ thực chất không vẽ dây dẫn thật (not actually routed), mà chỉ dự đoán nhu cầu đi dây ngang/dọc bên trong từng GCELL để nhận diện sớm các điểm nóng tắc nghẽn (congestion hotspots).
    
    - **Câu lệnh (Command):** Quá trình định tuyến phân tích này có thể được gọi chạy độc lập bằng lệnh `route_early_global`.

### 3. Đánh giá tính khả thi: Congestion Metrics (Trang 26)

Kết quả dự báo từ khâu Global Routing sẽ được lượng hóa thành con số để kỹ sư theo dõi và đưa ra quyết định.

- **Overflow (%) (Tỉ lệ tràn):** Đây là thước đo sống còn để đánh giá khả năng đi dây (routability) của thiết kế. Nó xuất hiện khi nhu cầu đi dây vượt quá giới hạn tài nguyên đang có. Tỉ lệ tràn càng thấp, khả năng đi dây thành công càng cao.
    
- **Định mức đánh giá:** * Nếu **Overflow < 1%**: Thiết kế rất an toàn, nhìn chung là có thể định tuyến thành công (generally routable).
    
    - Nếu **Overflow > 1%**: Cảnh báo rủi ro, thiết kế rất có khả năng sẽ gặp bế tắc và thách thức khi đi dây thật (likely routing challenges).
        
- **Phân tích Log thực tế:** Trong bảng log `Congestion Analysis`, phần mềm thống kê tỉ lệ Overflow chi tiết cho từng lớp kim loại theo cả hai chiều ngang (H) và dọc (V). Việc công cụ trả về kết quả `0.00% H` và `0.00% V` là minh chứng cho thấy khâu Global Placement đã rải cell cực kỳ hợp lý, không để xảy ra bất kỳ sự quá tải tài nguyên nào.

## Detailed Placement

### 1. Mục tiêu và Nguyên lý của Detailed Placement (Trang 27)

Sau khi thiết kế trải qua giai đoạn rải cell thô (Global Placement), **Detailed Placement** (sắp xếp chi tiết) sẽ được thực hiện để tinh chỉnh vị trí các phần tử vật lý:

- **Legalization (Hợp pháp hóa):** Khác với bước trước đó, ở giai đoạn này, các **Standard Cells** bắt buộc phải được căn chỉnh (aligned) khớp hoàn toàn vào các **Placement Rows** và các **Legal Sites**. Quá trình này đảm bảo tuyệt đối không còn bất kỳ sự chồng lấn (no overlaps) nào giữa các cell.
    
- **Tối ưu hóa cục bộ:** Các cell có kết nối tín hiệu với nhau chặt chẽ (strongly connected) sẽ được thuật toán kéo lại gần nhau hơn nhằm giảm thiểu độ trễ (reduce delay) và cải thiện các chỉ số **PPA** (Power, Performance, Area).
    
- Quá trình tinh chỉnh vị trí này được công cụ tính toán dựa trên nhiều yếu tố: chiến lược sắp xếp (placement strategies), ràng buộc thời gian (timing requirements), độ kết nối (net connectivity), tối ưu chiều dài dây (wirelength optimization), và kiểm soát kẹt dây (congestion awareness). Mục tiêu cuối cùng là tạo ra một layout hợp pháp, sẵn sàng cho các bước làm cây đồng hồ (CTS) và đi dây (Routing).

### 2. Thực thi bằng Câu lệnh và Phân tích Log (Trang 29)

Để chạy quá trình sắp xếp này, kỹ sư sử dụng câu lệnh `place_detail` hoặc `place_design` trên giao diện phần mềm (như Innovus). Quá trình tool hoạt động được ghi nhận qua Log:

- **Giới hạn không gian đi dây:** Log liên tục hiển thị cảnh báo `Max route layer is changed from 127 to 6 because there is no routing track above this layer`. Điều này cho thấy phần mềm nhận diện không có lưới đi dây (**Routing Track**) ở các lớp kim loại cao hơn 6, do đó nó tự động giới hạn không gian định tuyến.
    
- **Quá trình dịch chuyển Cell (Move report):** Để đạt được sự hợp pháp hóa (legalization) mà không bị chồng lấn, công cụ bắt buộc phải dời các cell sang các vị trí trống. Log ghi nhận `Detail placement moves 5551 insts` (đã dịch chuyển 5551 khối logic).
    
- Khoảng cách dời cell trung bình (mean move / mean displacement) khá nhỏ, ở mức $1.70 \mu m$. Tuy nhiên, có những cell bị đẩy đi rất xa với mức dịch chuyển lớn nhất (max move / max displacement) lên tới $40.75 \mu m$.
### 3. Đánh giá Chất lượng sau Detailed Placement (Trang 28)

Sau khi chạy xong, kỹ sư sẽ đối chiếu báo cáo tổng hợp (Results Example) để xem thiết kế có hội tụ và đạt chuẩn hay không:

- **Mức độ hợp pháp (Legalization Results):** Thiết kế đạt kết quả tuyệt đối với `0 cell overlaps` (không còn chồng chéo) và `0 site violations` (không vi phạm căn chỉnh lưới). **Row Utilization** (độ sử dụng hàng) ở mức 68.2%, chừa lại khoảng 31.8% không gian trống (whitespace). Độ dịch chuyển trung bình lúc này chỉ là $0.89 \mu m$.
    
- **Cải thiện Thời gian (QoR Improvement):** Việc kéo các cell lại gần nhau ở trang 27 đã mang lại tác dụng. Chỉ số **WNS** (Worst Negative Slack - thời gian trễ xấu nhất) được kéo giảm từ -212ps xuống mức tốt hơn là -148ps. Tổng độ trễ **TNS** (Total Negative Slack) trên toàn chip cũng cải thiện đáng kể từ -49ns xuống -28ns.
    
- **Cải thiện Tắc nghẽn (Congestion Improvement):** Sự phân bổ lại cell một cách hợp lý giúp giảm các điểm nóng kẹt dây (Congestion hotspots) từ 7 xuống chỉ còn 3. Đặc biệt, tỷ lệ tràn tài nguyên định tuyến (Maximum overflow) rớt mạnh từ mức rất cao 135.7% xuống còn 81.4%.