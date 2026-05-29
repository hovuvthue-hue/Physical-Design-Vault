# 1. Khởi tạo Mặt bằng và Các Thông số Giới hạn (Foundations & Control Parameters)

Quy hoạch mặt bằng (Floorplanning) là bước chuyển giao mang tính vĩ mô đầu tiên từ không gian logic (Netlist) sang không gian vật lý (Silicon). Sự thành bại của các khâu như Placement (sắp xếp linh kiện) và Routing (đi dây) phía sau phụ thuộc hoàn toàn vào chất lượng của bản quy hoạch này.

### 1.1. Bản chất và Ranh giới của Floorplanning

Quy hoạch mặt bằng là quá trình giải quyết bài toán không gian nhằm thỏa mãn các ràng buộc về diện tích, hiệu suất thời gian (Timing) và khả năng phân phối nguồn điện (Power).

- **Đầu vào (Inputs):** Danh sách kết nối mức cổng đã được tổng hợp (Gate-level Netlist), Tệp quy tắc công nghệ (Tech LEF), Ràng buộc thời gian (SDC), và Yêu cầu diện tích/nguồn từ kiến trúc hệ thống.
    
- **Đầu ra (Outputs):** Bản thiết kế xác định rõ: Chu vi lõi chip (Die/Core Area), Tọa độ I/O Pad, Tọa độ các khối IP/Macros cỡ lớn, và Cấu trúc mạng lưới phân phối nguồn sơ bộ (Power Grid).

### 1.2. Tiền xử lý Dữ liệu: Độc bản hóa Danh sách kết nối (Netlist Uniquification)

Đây là một khái niệm tối quan trọng được Tiến sĩ Adi Teman nhấn mạnh để ngăn chặn các lỗi tối ưu hóa tai hại.

Trong mã RTL, kỹ sư thường tái sử dụng code bằng cách khởi tạo (instantiate) một Module nhiều lần. Ví dụ: Khối `ALU` được gọi hai lần thành `ALU_1` và `ALU_2`. Ở mức logic, chúng chia sẻ chung một tệp định nghĩa.

Tuy nhiên, trong không gian vật lý, `ALU_1` và `ALU_2` sẽ được đặt ở hai góc khác nhau của vi mạch, chịu nhiễu và độ trễ đường dây hoàn toàn khác nhau.

- **Vấn đề:** Nếu không "Độc bản hóa", khi công cụ phát hiện `ALU_1` bị vi phạm Setup Time và tự động chèn thêm một bộ đệm (Buffer) để sửa lỗi, nó sẽ ngầm định áp dụng thay đổi đó cho cả `ALU_2` (nơi có thể không hề bị lỗi, làm hỏng luôn timing của `ALU_2`).
    
- ==**Giải pháp (Uniquification):** Phần mềm sẽ tạo ra các bản sao vật lý độc lập trong bộ nhớ (ví dụ: đổi tên thành `ALU_module_A` và `ALU_module_B`), cắt đứt mối liên kết dùng chung tài nguyên để công cụ tối ưu hóa có thể tác động cục bộ một cách an toàn.==

### 1.3. Các Thông số Kiểm soát Không gian (Spatial Control Parameters)

Khi khởi tạo mặt bằng (Initialize Floorplan), kỹ sư phải khai báo hai thông số toán học định hình hình dáng của vi mạch:

**A. Tỷ lệ khung hình (Aspect Ratio - AR):**

Quy định hình dáng của lõi vi mạch, thường được tính bằng tỷ lệ giữa Chiều cao và Chiều rộng.

- **Công thức:** $AR = \frac{Height}{Width}$
    
- _Ý nghĩa:_ Nếu $AR = 1$, vi mạch có hình vuông (tối ưu nhất cho các thuật toán bố trí). Nếu $AR = 0.5$, vi mạch có hình chữ nhật nằm ngang.

**B. Mật độ sử dụng lõi (Core Utilization - U):**

Chỉ số này biểu thị phần trăm diện tích lõi (Core Area) bị chiếm dụng bởi các linh kiện logic thực tế.

- **Công thức:** $U = \frac{\text{Area of Standard Cells} + \text{Area of Macros}}{\text{Total Core Area}}$
    
- _Chiến lược thiết lập:_ Không một kỹ sư nào thiết lập $U = 100\%$. Mức khởi điểm tiêu chuẩn công nghiệp thường nằm ở **70% - 80%**. Phần diện tích trống (White space) còn lại là bắt buộc để chừa không gian cho:
    
    1. Lưới đi dây kim loại (Routing tracks) tránh gây tắc nghẽn (Congestion).
        
    2. Diện tích dự phòng cho công cụ chèn thêm Buffers hoặc phóng to kích thước Cell (Upsize) trong quá trình sửa lỗi Timing ([[Spare cells|ECO]]) ở các bước sau.

### 1.4. Phân loại Giới hạn Diện tích Vi mạch

Tùy thuộc vào bản chất của thiết kế, kiến trúc sư vi mạch sẽ phân loại chip thành hai trường hợp để có chiến lược tối ưu hóa diện tích phù hợp:

- **Vi mạch Giới hạn bởi Lõi (Core-Limited Chip):** Xảy ra ở các vi mạch xử lý tính toán cực mạnh (như CPU, GPU). Lượng Standard Cell và Macro bên trong quá lớn, quyết định toàn bộ kích thước của Die. Số lượng chân cắm I/O ngoại vi ít và có thể dễ dàng xếp vừa vặn trên chu vi của chip.
    
- **Vi mạch Giới hạn bởi Pad (Pad-Limited Chip):** Xảy ra ở các vi mạch giao tiếp (như Router, Switch). Mạch logic bên trong rất ít, nhưng vi mạch lại cần hàng trăm chân I/O Pad để kết nối với bo mạch ngoài. Do các I/O Pad có kích thước vật lý khổng lồ và không thể thu nhỏ theo định luật Moore, chu vi của chip bắt buộc phải phình to để chứa đủ Pad, dẫn đến vùng lõi bên trong bị dư thừa diện tích rất nhiều (lãng phí tài nguyên silicon).

![[Pasted image 20260507212852.png]]

**Tiểu kết Phần 1:** Việc thiết lập sai Aspect Ratio hoặc ép Core Utilization lên quá cao ngay từ đầu sẽ dẫn đến những điểm tắc nghẽn vật lý (Congestion Hotspots) không thể cứu vãn ở giai đoạn Routing, buộc kỹ sư phải đập bỏ và chạy lại luồng thiết kế từ con số 0.
# 2. **Xử lý Vùng Biên và Các Linh kiện Đặc thù (I/O, Power & Pre-placed Cells)**

Sau khi đã chốt được kích thước tổng thể và tỷ lệ khung hình của lõi chip ở Phần 1, kỹ sư Physical Design không thể ngay lập tức ném các cổng logic (Standard Cells) vào trong. Chúng ta phải thiết lập một "bộ khung tĩnh" bao gồm các chân giao tiếp ngoại vi và các linh kiện vật lý bắt buộc để đảm bảo chip có thể được sản xuất và cấp nguồn an toàn.

### 2.1. Quy hoạch Vòng đệm Giao tiếp (I/O Pad Placement)

Các I/O Pad (chân cắm ngoại vi) là cầu nối duy nhất giữa thế giới vi mô bên trong vi mạch và thế giới vĩ mô của bo mạch (PCB) bên ngoài.

- **Nghịch lý định luật Moore:** Dù các transistor bên trong lõi chip có thể thu nhỏ xuống mức 5nm hay 3nm, các I/O Pad hầu như **không thể thu nhỏ (Do not scale with Moore's law)**. Nguyên nhân là do chúng bị giới hạn bởi công nghệ đóng gói cơ học (ví dụ: máy hàn dây - Wire bonding) và phải chịu tải điện dung/điện cảm cực lớn từ môi trường ngoài.
    
- **Chiến lược sắp xếp:** * I/O Pads thường được bố trí thành một vòng khép kín (I/O Ring) ở sát chu vi của Die.
    
    - Vòng khép kín này không chỉ truyền tải dữ liệu mà còn đóng vai trò là vành đai phân phối nguồn điện (Power distribution) mạnh mẽ nhất, dẫn điện áp cao từ ngoài vào và hạ áp (step-down) trước khi cấp cho lõi chip.
        
    - Vị trí của từng Pad phải được thống nhất chặt chẽ với đội ngũ thiết kế hệ thống (System/Board Designers) để khi đóng gói (Packaging), các dây cáp không bị chéo nhau.

### 2.2. Sắp xếp các Cell Tiền định (Pre-placed Cells / Physical-only Cells)

Đây là một nhóm các linh kiện cực kỳ đặc biệt. Chúng **không chứa bất kỳ hàm boolean logic nào** (không có mặt trong file Netlist ban đầu), nhưng bắt buộc phải được chèn vào mặt bằng vì lý do tính toàn vẹn vật lý và an toàn điện học.

Kỹ sư phải ra lệnh cho công cụ phân bổ 3 loại Cell Tiền định sau vào lưới mặt bằng (Placement Grid) trước khi làm bất cứ việc gì khác:

**A. Well-Tap Cells (Tế bào nối đế):**

- **Mục đích:** Ngăn chặn hiện tượng chập mạch chốt (Latch-up Effect). Trong cấu trúc CMOS, sự sắp xếp của các lớp N-well và P-substrate vô tình tạo ra các transistor lưỡng cực (BJT) ký sinh BPN/PNP. Nếu có nhiễu điện áp, các BJT này có thể kích hoạt, tạo ra dòng điện đoản mạch khổng lồ nung chảy vi mạch.
    
- **Giải pháp:** Tap Cells được chèn vào để nối trực tiếp N-well lên nguồn ($V_{DD}$) và P-sub xuống đất ($V_{SS}$), xả bỏ các dòng rò ký sinh. Kỹ sư phải tuân thủ nghiêm ngặt quy tắc khoảng cách tối đa giữa 2 Tap Cells theo cẩm nang nhà máy đúc (DRC Manual).

**B. End-Cap Cells / Boundary Cells (Tế bào giới hạn biên):**

- **Mục đích:** Đóng vai trò như "chốt chặn" ở hai đầu mút của mỗi hàng đặt linh kiện (Placement Row).
    
- **Tác dụng:** Trong quá trình sản xuất quang khắc (Lithography) và cấy ion, các Standard Cells nằm ở rìa hàng thường bị sai số hình học do thiếu linh kiện lân cận làm bệ đỡ. End-cap cells lấp vào các vị trí rìa này để bảo vệ các cổng logic thực sự nằm bên trong. Đồng thời, chúng ngắt mạch an toàn cho các thanh ray nguồn (Power rails) ở cuối hàng.

**C. Decap Cells (Tụ điện Tách sóng - Decoupling Capacitors):**

- **Mục đích:** Chống lại hiện tượng sụt áp động (Dynamic IR-Drop). Khi hàng triệu Flip-flop chuyển trạng thái cùng lúc tại sườn xung nhịp, chúng rút một dòng điện khổng lồ. Mạng phân phối nguồn có thể không đáp ứng kịp do điện cảm ký sinh của dây dẫn kim loại xa.
    
- **Giải pháp:** Kỹ sư sẽ nhồi các Decap Cells vào các khoảng trống (white space) gần các khối logic tiêu thụ nhiều điện. Chúng hoạt động như các "bể chứa điện cục bộ", xả dòng tức thời để giữ cho điện áp VDD luôn ổn định.

---
**Tiểu kết Phần 2:**
Việc thiết lập I/O và Pre-placed cells giống như việc bạn xây tường rào, đặt móng và đi các trụ chống sét trước khi xây nhà. Nếu quên Tap Cells, chip sẽ chết vì Latch-up. Nếu quên Decap, chip sẽ chạy sai ở tần số cao do sụt nguồn.

# 3. **Nghệ thuật Đặt Hard Macros & Giải quyết Tắc nghẽn (Macro Placement & Congestion)**

Trong giai đoạn Floorplan, các Hard Macros (như SRAM, ROM, PLL, hoặc các khối IP đúc sẵn) là những thực thể chiếm diện tích khổng lồ và không thể bị chia cắt. Việc bố trí chúng sai cách sẽ dẫn đến thảm họa không thể đi dây (Routing failure) ở các bước sau.

#### 3.1. Các Nguyên Tắc Vàng trong Macro Placement (Guidelines)

Vì các Hard Macros hoạt động như những "bức tường thành" cản trở lưới Routing, kỹ sư phải tuân thủ nghiêm ngặt các quy tắc sắp xếp sau:

- **Push to the Periphery (Đẩy ra vùng biên):**
    
    - Luôn đẩy các Macros ra sát các cạnh (edges) hoặc các góc (corners) của Core.
        
    - _Mục đích:_ Gom toàn bộ phần diện tích trống còn lại vào trung tâm thành một khu vực liên tục (Contiguous area). Các thuật toán Placement chỉ hoạt động đạt hiệu suất tối ưu khi chúng có một mặt bằng vuông vức, không bị đứt gãy để sắp xếp hàng triệu Standard Cells. Nếu bạn đặt Macro ở giữa Core, mặt bằng sẽ bị băm nát thành các hành lang hẹp, gây khó khăn cực độ cho thuật toán.
        
- **Pin Orientation (Định hướng chân cắm):**
    
    - Sử dụng tính năng hiển thị **Flylines** (Đường bay tín hiệu) trên phần mềm EDA để xem các chân (Pins) của Macro đang kết nối với I/O Pad nào hoặc Standard Cell nào.
        
    - Xoay (Rotate) hoặc Lật (Flip) Macro sao cho mặt chứa nhiều Pin hướng về phía trung tâm Core (nơi đặt Standard Cells). Tuyệt đối tránh việc áp mặt chứa Pin vào sát ranh giới Core vì khu vực đó không có không gian cho Routing.
        
    - _Lưu ý công nghệ:_ Ở các node tiến trình nhỏ (FinFET), việc xoay Macro 90 độ thường bị cấm ngặt nghèo do giới hạn quang khắc (Lithography rules) yêu cầu các lớp Poly phải chạy cùng chiều.

#### 3.2. Tính Toán Khoảng Cách Kênh (Channel Width Calculation)

Khi bắt buộc phải đặt hai Macros nằm cạnh nhau, kỹ sư không được đặt chúng dính sát vào nhau. Cần phải chừa lại một khoảng trống ở giữa, gọi là **Routing Channel**, để cho phép các Net từ Standard Cells đi xuyên qua hoặc kết nối vào chính các Macros đó.

- **Định lượng độ rộng Channel:** Không có con số ngẫu nhiên. Chiều rộng của kênh phải được tính toán dựa trên năng lực Routing của công nghệ.
    
- **Công thức ước lượng cơ sở:**$$Channel\_Width \approx \frac{\text{Total Pins} \times \text{Metal Pitch}}{\text{Available Routing Layers}}$$
    - **Total Pins:** Số lượng tín hiệu cần đi qua kênh này.
        
    - **Metal Pitch:** Bước kim loại (khoảng cách tối thiểu từ tâm dây này đến tâm dây kia) theo tệp công nghệ LEF.
        
    - **Available Routing Layers:** Số lớp Metal được phép dùng để đi dây trong khu vực đó.

#### 3.3. Phân Tích và Xử Lý Tắc Nghẽn Cục Bộ (Local Congestion Analysis)

Việc sắp xếp Macro định hình dòng chảy của dữ liệu (Dataflow). Nếu dòng chảy này bị ép qua một khu vực quá nhỏ, hiện tượng **Congestion** (Tắc nghẽn) sẽ xảy ra.

- **Nhận diện Hotspots:** Các khu vực chứa nhiều cổng logic có số lượng Pin dày đặc (ví dụ: các bộ MUX lớn) rất dễ trở thành tâm điểm tắc nghẽn.
    
- **Công cụ chẩn đoán (Trial Route / Global Route):** * Trong bước Floorplan, kỹ sư không chạy Detailed Routing (quá tốn thời gian). Thay vào đó, họ chạy **Trial Route** để phần mềm đưa ra ước lượng sơ bộ về mật độ dây dẫn.
    
    - Kết quả trả về là một bản đồ nhiệt (**Congestion Map / Heatmap**). Nếu xuất hiện các đốm màu đỏ hoặc trắng (chỉ số mật độ vượt quá 100% dung lượng track), kỹ sư bắt buộc phải can thiệp.
        
- **Kỹ thuật giải tỏa Congestion trong Floorplan:**
    
    1. Mở rộng các Routing Channels giữa các Macros.
        
    2. Dời các I/O Pads hoặc Macros ra xa nhau để phân tán luồng Net.
        
    3. Áp dụng các lệnh giới hạn mật độ Standard Cell (như Partial Placement Blockage) tại khu vực Hotspot để ép tool Placement tản bớt các Cell ra chỗ khác.

---
**Tiểu kết Phần 3:**

Macro Placement là một công việc mang tính "thủ công" và đòi hỏi tư duy hình học rất lớn từ kỹ sư. Một bản Floorplan với Macro Placement tồi sẽ khiến công cụ P&R báo lỗi thiếu Routing Track (Shorts, DRC violations) ở giai đoạn cuối, làm lãng phí hàng tuần lễ chạy máy.

# 4. Công cụ Quản lý Không gian (Blockages, Halos & Placement Regions)

Mặc dù các thuật toán tự động của phần mềm EDA rất thông minh, chúng thiếu đi trực giác vĩ mô của một kỹ sư. Nếu để công cụ tự do hoàn toàn, các Standard Cells có thể bị rải rác một cách vô tổ chức hoặc hội tụ quá mức gây nghẽn mạng lưới đi dây. Do đó, kỹ sư cần sử dụng các công cụ ranh giới để định hướng và "ép khuôn" không gian vật lý.

### 4.1. Vùng đệm Bảo vệ (Halos / Keep-out Margins)

Khi bố trí các Hard Macros (như SRAM), kỹ sư bắt buộc phải vẽ một vành đai bảo vệ xung quanh nó gọi là **Halo**.

- **Mục đích:** Bất kỳ Macro nào cũng có hàng trăm Pins nằm ở rìa để giao tiếp. Nếu công cụ đặt các Standard Cells dính sát vào vách của Macro, không gian đi dây kim loại (Routing tracks) trỏ vào các Pins này sẽ bị chặn đứng, dẫn đến lỗi đoản mạch (Shorts) hoặc vi phạm khoảng cách (Spacing DRC violations).
    
- **Đặc tính:** Halo áp đặt lệnh cấm hoàn toàn, không cho phép bất kỳ Standard Cell nào được đặt vào vành đai này. Tuy nhiên, nó vẫn cho phép dây dẫn (Nets) bay phía trên (Over-the-cell routing) ở các lớp Metal cao hơn.

### 4.2. Vùng cấm Đặt linh kiện (Placement Blockages)

Bên cạnh Halos được gắn liền với Macros, kỹ sư có thể tự do vẽ các hình khối không gian ở bất kỳ đâu trên Core và gán cho chúng các thuộc tính cấm (Blockages) để giải quyết hiện tượng Congestion:

- **Hard Blockage:** Lệnh cấm tuyệt đối. Không một Standard Cell nào được phép đặt vào khu vực này trong toàn bộ quy trình thiết kế.
    
- **Soft Blockage:** Lệnh cấm linh hoạt. Trong pha Placement ban đầu, công cụ không được phép xếp các Standard Cells logic vào đây. Tuy nhiên, ở các pha tối ưu hóa sau đó (In-Place Optimization / ECO), công cụ được quyền chèn thêm các bộ đệm (Buffers/Inverters) vào khu vực này để sửa các vi phạm Timing (Setup/Hold).
    
    - _Ứng dụng:_ Thường được dùng ở các hành lang hẹp giữa Macro và ranh giới Core.
        
- **Partial Blockage:** Lệnh giới hạn mật độ. Thay vì cấm 100%, kỹ sư thiết lập một mức trần Utilization (ví dụ: 40%). Công cụ vẫn được phép đặt Standard Cells vào, nhưng phải sắp xếp thưa thớt.
    
    - _Ứng dụng:_ Cực kỳ hiệu quả để rã đám các điểm nóng tắc nghẽn (Congestion Hotspots) sinh ra bởi các khối logic có mật độ kết nối chéo cao như MUX hoặc Multipliers.

### 4.3. Phân vùng Định hướng Logic (Placement Regions)

Đôi khi, sơ đồ Dataflow yêu cầu toàn bộ các cổng logic thuộc cùng một Module (ví dụ: Module giải mã Audio) phải nằm cụm lại với nhau để giảm thiểu Net Delay. Kỹ sư sẽ sử dụng Placement Regions để ánh xạ cây phân cấp logic (Logical Hierarchy) lên tọa độ vật lý. Các công cụ này được chia thành 4 cấp độ từ lỏng lẻo đến cực đoan:

1. **Soft Guide:** Cấp độ lỏng nhất. Phần mềm được khuyến khích gom cụm (cluster) các Cells của module lại với nhau, nhưng không có bất kỳ tọa độ vật lý bắt buộc nào.
    
2. **Guide:** Kỹ sư vẽ một ranh giới cụ thể. Công cụ sẽ cố gắng nhét các Cells mục tiêu vào vùng này. Nếu vùng bị đầy, Cells có quyền tràn ra ngoài. Các Cells lạ (không thuộc module) cũng có quyền đi vào vùng này.
    
3. **Region:** Lệnh bắt buộc. 100% các Cells của module mục tiêu **phải** nằm trọn trong ranh giới này, không được phép tràn ra. Tuy nhiên, nếu vùng còn khoảng trống, các Cells lạ vẫn được phép xen kẽ vào.
    
4. **Fence (Hàng rào ranh giới):** Cấp độ kiểm soát độc quyền và tàn bạo nhất. Toàn bộ Cells mục tiêu bắt buộc nằm bên trong, và **tuyệt đối cấm** bất kỳ Cell lạ nào xâm nhập vào vùng Fence này.

_Lưu ý chuyên môn:_ Trong thực tế, kỹ sư Physical Design dày dạn kinh nghiệm rất hạn chế lạm dụng **Region** và **Fence**. Việc ép buộc thuật toán quá mức thường phá vỡ tính hội tụ của Timing và tạo ra Congestion giả tạo. Các công cụ này chỉ nên dùng khi bạn nắm chắc 100% hướng chảy của dữ liệu (Dataflow) trên vi mạch.