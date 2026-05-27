# 1. Standard Cell Placement

## Trang 6
Tại sao việc định nghĩa chuẩn xác khu vực này lại quan trọng:
- **Enables legal cell placement (Đảm bảo sắp xếp hợp lệ):** **Standard Cells** không thể bị ném vào chip một cách lộn xộn. Chúng bắt buộc phải được bám vào các vị trí lưới (legal sites) và xếp thành từng hàng (row structure) theo đúng thông số công nghệ mà nhà máy đúc (Foundry) yêu cầu.
    
- **Supports efficient utilization (Tối ưu hóa mật độ):** Đảm bảo vùng lõi được sử dụng với hiệu suất cao nhất nhưng không vi phạm các luật vật lý của nhà máy đúc.
    
- **Facilitates DRC-lean routing (Hỗ trợ định tuyến sạch):** Bằng cách bảo lưu đủ không gian, nó đảm bảo các dây tín hiệu (signal), dây nguồn (power) và dây xung nhịp (clock) khi được nối (Routing) sẽ không bị chập chéo hay vi phạm luật thiết kế (**DRC**).
    
- **Facilitates Timing Closure (Hỗ trợ hội tụ thời gian):** Một không gian gom cụm tốt giúp rút ngắn đường đi của dòng dữ liệu (Dataflow), giảm độ trễ (**Net Delay**) và giúp vi mạch dễ dàng vượt qua các bài kiểm tra **Timing**.

Key Concepts:
- **Standard Cell Placement Area:** Tổng khu vực trống có thể đặt linh kiện.
    
- **Placement Rows:** Các dải hàng ngang vô hình để gài các ô chuẩn vào (giống như các hàng gạch).
    
- **Wiring Tracks:** Lưới các đường ray dùng làm xa lộ đi dây kim loại.
    
- **Placement Blockages & Routing Blockages:** Các khu vực "vùng cấm" do kỹ sư chủ động vẽ ra để cấm phần mềm rải **Standard Cells** hoặc cấm đi dây đi ngang qua, nhằm né các điểm nóng tắc nghẽn (**Congestion Hotspots**).

## Trang 7: Standard Cell Placement Area / Rows

**Placement Rows:** Bên trong vùng lõi, không gian được chia thành các dải ngang cách đều nhau gọi là **Placement Rows**. Đây là các khu vực không gian hợp lệ (legal regions) duy nhất cho phép sắp xếp **Standard Cells**.

### Lưới Cơ sở (Sites) và Đặc tính Hình học của Cell

- **Sites:** Mỗi **Placement Row** được cấu thành từ vô số các **Sites** hợp lệ. Các ô lưới này được định nghĩa với kích thước và quy tắc căn chỉnh khớp tuyệt đối với yêu cầu của thư viện **Standard Cell**.
    
- **Cell Height (Chiều cao đồng nhất):** Chiều cao của mọi cổng logic (Cell) luôn là một mức đồng nhất (Uniform Height) và bắt buộc phải là bội số của lưới định tuyến ngang (Horizontal routing grid).
    
- **Cell Width (Chiều rộng linh hoạt):** Chiều rộng của linh kiện có thể thay đổi tùy thuộc vào độ phức tạp của logic bên trong, nhưng bắt buộc phải là bội số của lưới định tuyến dọc (Vertical routing grid / Site Width).
    
- **Cell Pins:** Vị trí chân cắm của linh kiện được khuyến nghị thiết kế nằm ngay tại các điểm giao cắt của cả hai lưới định tuyến ngang và dọc để tối ưu hóa khả năng tiếp cận khi đi dây (Routing).
  
- **Cell Origin (Điểm gốc tọa độ của Cell):** Đây là một điểm tham chiếu (thường là góc dưới cùng bên trái) của một **Standard Cell**. Khi phần mềm muốn cầm một Cell để đặt vào mặt bằng chip, nó không cầm bừa bãi mà sẽ tóm lấy điểm **Cell Origin** này và đặt nó khớp chính xác vào một tọa độ trên lưới.
  
- **Cell PR Boundary (Ranh giới không gian Place & Route):** Đây là một "chiếc hộp vô hình" (Bounding Box) bao bọc toàn bộ kích thước vật lý của một **Standard Cell**. Phần mềm EDA sử dụng **PR Boundary** để nhận biết Cell này to chừng nào, chiếm bao nhiêu diện tích, từ đó đảm bảo rằng khi xếp các Cell cạnh nhau, chúng không bị đè lên nhau (overlap) hoặc vi phạm luật khoảng cách.

### Row Orientation & Abutment
Đây là kỹ thuật vật lý kinh điển và quan trọng nhất được minh họa trên Trang 7 nhằm tiết kiệm không gian định tuyến và chia sẻ tài nguyên nguồn (Power/Ground)

Các **Placement Rows** không được xếp theo một hướng duy nhất mà luân phiên đảo chiều liên tục (Alternates) để kích hoạt tính năng lật linh kiện (Cell flipping):

- **R0 Orientation (Hướng nguyên bản):** Hướng đứng thẳng (Upright), không xoay, không lật gương. Tại hướng này, thanh ray nguồn **VDD** nằm ở mép trên cùng và **VSS** nằm ở mép dưới cùng của Cell.
    
- **MY Orientation (Hướng lật trục Y):** Hàng bị lật ngược từ trên xuống dưới (Flip top to bottom). Việc lật này khiến ray **VSS** chuyển dịch lên trên và **VDD** bị đưa xuống dưới.
    
- **Sự tiếp giáp hoàn hảo (Abutment):** Việc luân phiên xếp hàng xen kẽ (R0 rồi đến MY) giúp cho đường **VDD** của hàng này áp sát liền mạch với đường **VDD** của hàng kế tiếp (Abutted PG Rail). Tương tự, các giếng bán dẫn **N-Well** và thanh ray **VSS** của hai hàng lân cận cũng chạm sát và hợp nhất vào nhau (Abutted N-Well).

### Liên Kết:

- **Thiết lập cấu trúc hàng (Row Orientation):** Trước khi đặt Cell, phần mềm P&R khởi tạo `Placement Rows`. Tại bước này, thuộc tính `Row Orientation` được xác định để quy định hướng vật lý của từng hàng. Thay vì để tất cả các hàng cùng một hướng, thuật toán sẽ thiết lập chúng theo cơ chế xen kẽ (alternating).
    
- **Kỹ thuật lật hàng (R0 và MY Orientation):**
    
    - Hàng thứ nhất được thiết lập `R0 Orientation`: Đây là hướng chuẩn, trong đó thanh ray nguồn VDD nằm ở mép trên và VSS nằm ở mép dưới của hàng.
        
    - Hàng thứ hai liền kề được thiết lập `MY Orientation`: Hàng bị lật ngược (mirrored) theo trục Y. Kết quả là vị trí các thanh ray nguồn bị đảo ngược: VSS lên trên và VDD xuống dưới.
        
- **Hợp nhất tài nguyên (Abutted PG Rail và N-well):** Sự kết hợp giữa `R0 Orientation` của hàng dưới và `MY Orientation` của hàng trên tạo ra hiện tượng **Abutment** (tiếp giáp khít).
    
    - Thanh ray VDD của hàng dưới sẽ áp sát và hợp nhất với thanh ray VDD của hàng trên, tạo thành một `Abutted PG Rail` chung.
        
    - Tương tự, các vùng bán dẫn `N-well` của hai hàng cũng tiếp xúc và hợp nhất (`Abutted N-well`). Điều này giúp tiết kiệm 50% diện tích kim loại dành cho mạng lưới nguồn điện nội bộ.
        
- **Định vị linh kiện trên hạ tầng (Cell Origin và PR Boundary):** Khi hạ tầng hàng đã sẵn sàng, `Standard Cell` được đưa vào. Thuật toán gắn `Cell Origin` vào tọa độ `Site` trên hàng. `Cell PR Boundary` đảm bảo linh kiện chiếm dụng đúng chiều cao của hàng và khớp với các thanh ray nguồn đã được hợp nhất ở bước trước.
    
- **Hội tụ ranh giới hàng dọc (Multiple of Site Width):** Để duy trì tính liên tục của `Abutted PG Rail` chạy ngang suốt vi mạch, mọi Cell trong cùng một hàng phải kết thúc đúng tại mép của một `Site`. Quy tắc `Multiple of Site Width` đảm bảo rằng khi các Cell áp sát nhau (`Abutment` theo chiều ngang), không có bất kỳ khoảng hở nào làm đứt đoạn mạng lưới nguồn hoặc vùng `N-well` đã được quy hoạch.

## Trang 8: Wiring Tracks

### Surface Analysis
Xác định ranh giới hoạt động của hai quy trình cốt lõi là `Placement` và `Routing`.

**PR Boundary:** Là đường viền bao ngoài cùng, xác định tổng diện tích của một khối thiết kế.

**Standard Cell Placement Area:** Là vùng không gian khả dụng nằm bên trong `PR Boundary` (được biểu diễn bằng vùng màu đỏ trong hình). Nguyên tắc cứng được tài liệu nhấn mạnh là phần mềm (Tool) tuyệt đối không được phép đặt bất kỳ `Standard Cell` nào vượt ra bên ngoài ranh giới vùng đỏ này.

**Wiring Tracks:** Được định nghĩa là một tập hợp các đường lưới định tuyến (`routing grid lines`) được phân bổ với khoảng cách đều đặn trên mặt bằng. Nhiệm vụ duy nhất của các `Wiring Tracks` này là dẫn hướng (`guide`) để xác định chính xác các tọa độ hợp lệ mà phần mềm có thể vẽ đường dây tín hiệu (`wires`) trong quá trình `Routing`.

### Directional & Relational Analysis
Bóc tách cách thức các `Wiring Tracks` được tổ chức và kiểm soát hướng đi của tín hiệu.

**Preferred Routing Direction (Định hướng ưu tiên):** Mỗi lớp kim loại (`metal layer`) không được phép định tuyến tự do mọi hướng. Các `Wiring Tracks` bắt buộc phải tuân theo một hướng ưu tiên được thiết lập sẵn cho lớp kim loại đó.

- Hình ảnh minh họa chỉ rõ: Lớp kim loại `M2` (`M2 Track`) bị khóa theo hướng ngang (`horizontal`).
    
- Lớp kim loại `M3` (`M3 Track`) bị khóa theo hướng dọc (`vertical`).

**Kiến trúc lưới trực giao (Orthogonal Grid):** Sự kết hợp giữa `M2 Track` chạy ngang và `M3 Track` chạy dọc thiết lập một mạng lưới tọa độ trực giao ngay phía trên `Standard Cell Placement Area`. Hệ thống này buộc thuật toán `Routing` phải phân tách trục di chuyển của tín hiệu, nhằm loại bỏ hiện tượng đan chéo dây trên cùng một lớp kim loại.

### Design Rules & Physical Constraints Analysis

Bản chất toán học định hình nên các `Wiring Tracks` thông qua thông số `Track Pitch`.

Hình minh họa chỉ ra `M2 Pitch` là khoảng cách giữa hai `M2 Track` liền kề, và `M3 Pitch` là khoảng cách giữa hai `M3 Track` liền kề. `Track Pitch` là một hằng số công nghệ mang tính quyết định đến mật độ của lưới định tuyến.

Tài liệu định nghĩa toán học của thông số này bằng phương trình: **`Track Pitch = Metal Width + Spacing`**.
- **Metal Width:** Là chiều rộng cắt ngang vật lý tối thiểu của một sợi dây kim loại.
    
- **Spacing:** Là khoảng cách cách điện vật lý tối thiểu bắt buộc phải duy trì giữa hai sợi dây kim loại chạy song song.

Các giá trị `Width` và `Spacing` không do kỹ sư tự chọn mà được quy định nghiêm ngặt bởi `Design Rules` (Quy tắc thiết kế) và `Manufacturing Requirements` (Năng lực quang khắc của nhà máy đúc). Do đó, cấu trúc `Wiring Tracks` chính là sự phản ánh trực tiếp giới hạn vật lý của công nghệ sản xuất xuống không gian thiết kế của phần mềm.

### Liên kết:

Các yêu cầu sản xuất của nhà máy đúc (`Manufacturing Requirements` / `Design Rules`) quy định giới hạn vật lý bắt buộc của hai thông số cơ sở: `Metal Width` (độ rộng dây) và `Spacing` (khoảng cách cách điện). Sự cộng gộp của hai thông số này tạo thành `Track Pitch`. Tức là, chính công nghệ sản xuất đã quyết định toán học hóa khoảng cách giữa các `Wiring Tracks` trên mặt bằng.

Các `Wiring Tracks` sau khi được xác định khoảng cách bởi `Track Pitch` sẽ bị khóa cứng hướng đi theo `Preferred Routing Direction` của từng lớp kim loại. Sự xếp chồng giữa các lớp kim loại khác hướng nhau (cụ thể: `M2 Track` chạy theo hướng ngang cắt với `M3 Track` chạy theo hướng dọc) thiết lập nên một mạng lưới tọa độ trực giao (`Orthogonal Grid`).

Mạng lưới `Orthogonal Grid` này không lơ lửng mà được bao phủ trực tiếp lên vùng không gian hợp lệ dùng để đặt linh kiện, tức là `Standard Cell Placement Area` (nằm gọn bên trong `PR Boundary`). Nhờ sự liên kết từ dưới lên trên này, sau khi thuật toán hoàn tất việc xếp `Standard Cell` vào vùng lõi, thuật toán `Routing` sẽ chỉ việc men theo hệ thống `Wiring Tracks` đã thiết lập để đi dây, đảm bảo tuyệt đối không sinh ra lỗi chập mạch hay vi phạm `Design Rules`.

Toàn bộ trang 8 có thể tóm gọn lại thành chuỗi nguyên nhân - kết quả: **`Design Rules` -> `Track Pitch` -> `Preferred Routing Direction` -> `Orthogonal Grid` che phủ `Standard Cell Placement Area`.**

## Trang 9: Placement & Wiring Blockages

### Surface Analysis
Hai cơ chế cốt lõi để kiểm soát không gian thiết kế: `Placement Blockages` và `Routing Blockages`.
- **Placement Blockages:** Là các ranh giới được thiết lập để giới hạn hoặc cấm hoàn toàn việc sắp xếp các `cells` và `macros` trên mặt bằng thiết kế. Cơ chế này được chia thành 4 phân loại chính: `Hard`, `Partial`, `Soft`, và `Macro-only`.
    
- **Routing Blockages:** Là các vùng hạn chế do người thiết kế (designer-defined) định ra nhằm ngăn chặn hoặc giới hạn việc đi dây tín hiệu (`signal routing`).
    
- **Công cụ thực thi (Tool Commands):** Trang tài liệu cung cấp các đoạn mã lệnh trực tiếp (như `create_place_halo`, `create_placement_blockage`, `create_routing_blockage`) để khởi tạo các cấu trúc `blockages` này vào hệ thống thiết kế.

### Technical Rules Analysis
Bộ quy tắc kỹ thuật định nghĩa thuộc tính của từng loại `blockage`.

**Quy tắc của Placement Blockages:**
- **Hard:** Cấm hoàn toàn việc đặt bất kỳ `cells` nào vào khu vực này ("Block all cells here").
    
- **Partial:** Cho phép đặt `cells` nhưng bị giới hạn bởi mức `Utilization` tối đa (ví dụ: Max `Utilization` là 50%).
    
- **Soft:** Hoạt động như một cơ chế lọc, chỉ cho phép đặt các `cells` cụ thể phục vụ mục đích tối ưu hóa như buffers, inverters, clock gaters ("Only allow buffers here").
    
- **Macro-only:** Chỉ cấm đặt `macros` tại khu vực được chỉ định.

Khác với `Placement Blockages` chỉ hoạt động trên không gian bề mặt của mặt bằng, `Routing Blockages` can thiệp vào các `metal layers` cụ thể (ví dụ: chặn đi dây trên các lớp M2-M4). Mục đích của chúng là điều hướng `router` nhằm tránh các vùng nhạy cảm (`sensitive regions`), bảo toàn tài nguyên (`preserve resources`), hoặc bắt buộc hệ thống tuân thủ `design constraints`.

### Command & Spatial Analysis
Các câu lệnh được cung cấp để thấy cách `blockages` được ánh xạ vào mặt bằng vật lý.

- **Thiết lập khoảng đệm cục bộ (Halo):** Lệnh `create_place_halo -halo_deltas {17 17 17 17} -inst ROM` thiết lập một vùng cấm bao quanh 4 cạnh của một `macro` cụ thể (ở đây là `ROM` instance).
    
- **Định nghĩa vùng Placement Blockage:** Lệnh `create_placement_blockage -type hard -boundary {10 10 50 50}` gán thuộc tính cấm tuyệt đối (`hard`) lên một tọa độ hình chữ nhật được xác định bởi điểm tọa độ gốc và điểm tọa độ chéo.
    
- **Định nghĩa vùng Routing Blockage:** Lệnh `create_routing_blockage -layers {M2 M3 M4} -boundary {100 100 200 200}` minh họa việc kết hợp tọa độ bề mặt (`boundary`) với tham số trục dọc (`layers`). Hệ thống tạo ra một ranh giới cấm định tuyến chạy xuyên qua 3 lớp kim loại (`M2`, `M3`, `M4`) tại vùng tọa độ 100 100 200 200.
# Build Power Distribution Network (PDN)

Power Planning: Power Planning involves analyzing the chip power consumption and distributing power from the IO area to all the cells in the core area.

Purposes  
- Providing power to all cells on chip - All cells need power to operate  
- Minimizing IR Drops - Meeting maximum IR drop constraints - Helping timing closure  
- Minimizing self-heating - No EM rules violations 
- Avoiding over-designing - Minimizing impact on routability - Minimizing impact on die cost

## Trang 12: Power Distribution Structure
### Surface Analysis
Trang 12 định nghĩa rõ `Power Distribution Structure` (Cấu trúc phân phối nguồn) với nguyên tắc nền tảng: Mọi `cell` trong vi mạch đều bắt buộc phải được kết nối với mạng lưới `Power` và `Ground` (`PG`). Nguồn điện này có thể được cung cấp từ các `external sources` (nguồn bên ngoài chip) hoặc thông qua các `on-chip voltage regulators` (bộ điều áp trên chip). Hệ thống cũng có thể chứa nhiều hơn một `core voltage` (điện áp lõi).

Các thực thể hình học tạo nên mạng lưới cấp nguồn này bao gồm:

- **PG Rings:** Là các vòng kim loại rộng (`wide metal rings`) được thiết kế để bao quanh toàn bộ vùng `Core` hoặc bao quanh các `Macros`. Sơ đồ cũng ghi nhận sự hiện diện của `Core PG Rings` và `Macro (Block) PG Rings`.
    
- **PG Stripes / Straps:** Là các dải kim loại chạy theo hướng dọc (`vertical`) và ngang (`horizontal`) ngay bên trong vùng lõi để phân phối dòng điện.
    
- **Standard Cell PG Rails:** Là các đường ray cấp nguồn nằm trực tiếp bên trong mỗi `Standard Cell Row`.
    
- Ngoài ra, cấu trúc còn bao gồm các `PG Pads`, `I/O and Corner Pads` được đặt trên `Padframe` để tiếp nhận nguồn từ bên ngoài.

### Hierarchical Structure Analysis
tính liên kết thứ bậc của mạng lưới thông qua khái niệm `Hierarchical Structure`. Dòng điện không đi trực tiếp từ ngoài vào linh kiện mà được phân rã qua nhiều cấp độ không gian:

- **Cấp độ tiếp nhận:** Nguồn điện đi vào vi mạch thông qua các `Power Pad Connections` và các `I/O and Corner Pads`.
    
- **Cấp độ bao bọc:** Dòng điện sau đó được dẫn vào các `Core PG Rings` để ôm trọn lõi chip. Đối với các khối IP như `Macro Cell (RAM)`, chúng sẽ có ranh giới bảo vệ riêng biệt thông qua `Block Power Ring to Core Spacing`, `Block Halo` và `Macro (Block) PG Rings`.
    
- **Cấp độ phân phối trục chính:** Từ các `Rings`, dòng điện được đưa vào hệ thống lưới `PG Stripes/Straps` chạy đan xen xuyên suốt diện tích lõi.
    
- **Cấp độ tiêu thụ cục bộ:** Cuối cùng, năng lượng từ hệ thống `Stripes/Straps` sẽ được trích xuất xuống các `Standard Cell PG Rails` để thực hiện nhiệm vụ `Standard Cell Power Connections` trực tiếp cho các `Standard Cells`.

### Multi-Layer Distribution Analysis
Giải thích lý do vật lý đằng sau cách sắp xếp không gian này thông qua chiến lược `Multi-Layer Distribution`.

1. **Thiết lập lưới không gian:** Hệ thống sử dụng các `metal layers` khác nhau cho các hướng đi dây dọc và ngang để tạo thành một cấu trúc lưới (`mesh` hoặc `grid`) vững chắc.
    
2. **Tối ưu hóa điện trở và tắc nghẽn (Higher Layers):** Các `higher metal layers` (lớp kim loại cao, ví dụ: `M7`, `M8`) được chỉ định độc quyền để vận chuyển dòng điện tổng (`global power`). Bản chất vật lý của các lớp kim loại cao là có tiết diện lớn, giúp sinh ra ít điện trở hơn (`less resistance`) và đồng thời giải phóng không gian bên dưới để giảm thiểu tắc nghẽn định tuyến (`congestion`).
    
3. **Mạng lưới vi mô (Lower Layers):** Ngược lại, các `lower layers` (lớp kim loại thấp, ví dụ: `M1-M3`) lại được sử dụng để làm lưới phân phối cục bộ (`local distribution`) nhằm cắm trực tiếp xuống các `standard cells`.

**Tổng kết:** Thông qua 3 vòng phân tích liên tiếp, trang 12 cho thấy `Power Distribution Structure` là một mạng lưới toán học và vật lý chặt chẽ. Nó chuyển tiếp dòng điện từ ngoài vào trong thông qua một `Hierarchical Structure` (từ `Pads` -> `Rings` -> `Stripes` -> `Rails`) và tuân thủ các quy tắc phân bổ `Multi-Layer Distribution` nghiêm ngặt để tối ưu hóa `resistance` và `congestion`.

## Trang 13, 14: Power Nets Connections và Creating the VDD/VSS Rings