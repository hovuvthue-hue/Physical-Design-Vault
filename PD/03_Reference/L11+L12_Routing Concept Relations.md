# Quan Hệ Toàn Diện Giữa Các Khái Niệm Trong IC Physical Design Routing

Tài liệu này tổng hợp và diễn giải mối quan hệ giữa tất cả khái niệm xuất hiện trong Lecture 11 (Routing Part I) và Lecture 12 (Routing Part II), có đối chiếu với các nguồn học thuật và công nghiệp. Mọi mối quan hệ nhân quả, phụ thuộc, và ràng buộc đều được trình bày bằng văn bản thuần tuý.

---

## PHẦN I: FOUNDATION — NỀN TẢNG CỦA TOÀN BỘ ROUTING FLOW

### 1.1 Physical Design Flow Là Bối Cảnh Quyết Định Thứ Tự Phụ Thuộc

Routing không tồn tại độc lập. Nó là mắt xích thứ năm trong chuỗi Data Import, Floorplanning, Placement, Clock Tree Synthesis (CTS), Routing, và Data Export. Mỗi bước trước đó tạo ra nền tảng vật lý mà routing phải kế thừa, và mỗi bước sau đó phụ thuộc vào chất lượng của routing output. Floorplanning quyết định diện tích die, vị trí macro và IO pad, tạo ra khung không gian mà router phải làm việc bên trong. Placement xác định tọa độ của từng standard cell, từ đó quyết định pin location và khoảng cách giữa các cell cần kết nối. CTS phân phối clock đến toàn bộ Flip-flop, và kết quả CTS phải được bảo vệ sau khi routing bắt đầu vì clock net đã được route với đặc tính RC cụ thể. Việc Routing chạy tốt hay xấu phụ thuộc vào Placement và Floorplanning chất lượng thế nào, vì congestion trong routing phần lớn bắt nguồn từ mật độ cell cao hoặc floorplan không hợp lý.

Quan hệ ngược chiều cũng tồn tại: nếu timing sau Routing quá xấu, kỹ sư phải quay lại Placement hoặc thậm chí CTS để cải thiện, vì routing chỉ có thể recover một lượng nhất định timing margin. Điều này làm cho Physical Design Flow là một vòng lặp có phản hồi, không phải chuỗi tuyến tính.

### 1.2 PPA Là Tiêu Chí Xuyên Suốt Liên Kết Mọi Quyết Định Routing

Mọi quyết định trong routing đều tác động đồng thời đến ba chiều Power, Performance (Timing), và Area. Wire rộng hơn giảm điện trở nhưng tăng capacitance và chiếm thêm area. Via nhiều hơn tăng reliability nhưng tăng diện tích và có thể tăng congestion. NDR cải thiện Signal Integrity nhưng tiêu thụ gấp đôi routing resource trên mỗi net được áp dụng. Không có quyết định nào chỉ tác động một chiều. Ba chiều PPA là hệ quy chiếu duy nhất để đánh giá trade-off trong mọi bước.

---

## PHẦN II: TECHNOLOGY FILES — NGUỒN GỐC CỦA MỌI RÀNG BUỘC

### 2.1 LEF Là Bản Dịch Vật Lý Giữa Foundry Và EDA Tool

LEF (Library Exchange Format) là file trung gian quan trọng nhất giữa foundry và routing tool. Foundry biết chính xác quá trình fabrication của họ cho phép wire có width tối thiểu bao nhiêu, via cần enclosure bao nhiêu, và track pitch phải là bao nhiêu để đảm bảo yield. Tất cả kiến thức đó được mã hóa vào LEF. PnR tool không thể "biết" vật lý của từng technology node nếu không đọc LEF. Đây là lý do mọi routing action của tool đều phản chiếu trực tiếp các constraint trong LEF.

Technology LEF chứa hai nhóm thông tin có mối quan hệ nhân quả rõ ràng. Nhóm thứ nhất là định nghĩa routing layer với direction (Horizontal hoặc Vertical), Pitch, minWidth, spacing rule, và antenna model. Nhóm thứ hai là via rule với cut spacing, enclosure trên cả hai metal layer trên và dưới. Hai nhóm này kết hợp để định nghĩa hoàn toàn không gian hợp lệ mà router có thể đặt wire và via. Bất kỳ wire nào không tuân thủ các con số trong LEF đều tạo ra DRC violation.

Cell LEF bổ sung bằng thông tin về từng standard cell cụ thể: PR boundary, vị trí pin, layer nào pin tiếp xúc. Routing tool dùng Cell LEF để biết chính xác pin của từng cell instance nằm ở layer nào và tọa độ nào, từ đó mới có thể tiếp cận pin đúng cách trong quá trình detailed routing.

### 2.2 ITF Là Cầu Nối Giữa Vật Lý Fabrication Và RC Extraction

ITF (Interconnect Technology File) có quan hệ hoàn toàn khác với LEF. LEF cho router biết phải làm gì về mặt hình học. ITF cho RC extraction tool biết wire có các đặc tính điện gì sau khi fabricate. ITF định nghĩa resistance per unit length (Ω/μm) của từng metal layer, capacitance per unit area, capacitance bên (lateral coupling), capacitance dọc (vertical coupling), và fringe capacitance. Những thông số này là input bắt buộc để tính SPEF sau routing.

ITF và process corner có quan hệ một-nhiều: mỗi ITF file tương ứng với một corner cụ thể (Typical, Fast, Slow kết hợp với nhiệt độ và điện áp). Đây là lý do RC extraction tool cần nhiều ITF files để cover tất cả signoff corners. Nếu extraction dùng ITF corner không đúng, SPEF không chính xác, và STA sign-off sẽ pass ở corner sai trong khi fail ở corner thực tế, gây ra silicon failure.

### 2.3 SDC Là Ràng Buộc Timing Độc Lập Với Physical Implementation

SDC (Synopsys Design Constraints) chứa định nghĩa clock, input/output delay, multicycle path exceptions, và false path. SDC không mô tả vật lý nhưng quyết định target mà routing phải đạt. Timing-driven routing cần đọc SDC để biết path nào critical, net nào cần được route với ưu tiên cao. Sau khi routing hoàn thành và RC extraction cho ra SPEF, STA tool kết hợp SDC và SPEF để tính slack cho từng path. SDC là nguồn "yêu cầu" và SPEF là nguồn "thực tế" — STA so sánh hai nguồn này để phát hiện violation.

---

## PHẦN III: METAL STACK VÀ INTERCONNECT PHYSICS

### 3.1 Wire Geometry Quyết Định RC Và RC Quyết Định Delay

Đây là chuỗi nhân quả trung tâm của toàn bộ routing. Wire resistance R phụ thuộc vào vật liệu (điện trở suất ρ), chiều dài L, chiều rộng W, và độ dày T theo công thức R = ρL/(W×T). Wire capacitance C gồm ba thành phần: capacitance dọc với substrate và các layer trên/dưới (phụ thuộc diện tích mặt nằm ngang), coupling capacitance bên với wire kề nhau trên cùng layer (phụ thuộc chiều dài song song và khoảng cách), và fringe capacitance ở cạnh wire. Tổng RC tạo ra delay của wire xấp xỉ theo công thức Elmore delay là 0.38×R×C. Mọi quyết định routing về layer, width, spacing, và độ dài đều trực tiếp biến đổi R và C theo cách vật lý có thể tính toán được.

Delay tỷ lệ với L² vì khi L tăng, R tăng tuyến tính theo L và C cũng tăng tuyến tính theo L, nên RC tăng theo L². Đây là lý do wire dài là kẻ thù của timing, và router có xu hướng ưu tiên đường đi ngắn nhất cho critical net.

### 3.2 Metal Layer Hierarchy Phản Ánh Trade-off Resistance Và Routing Density

Metal stack được tổ chức thành hệ thống phân cấp với mục đích cụ thể cho mỗi tier. Layer thấp (M1, M2) có pitch nhỏ nhất, wire hẹp nhất, và được dùng cho local connection trong phạm vi gần. Layer cao hơn (M5, M6 trở lên) có wire dày hơn, pitch lớn hơn, và điện trở thấp hơn do tiết diện lớn hơn. Critical net và clock net được route trên layer cao vì resistance thấp giúp giảm delay và crosstalk (vì pitch lớn hơn nghĩa là wire có thể xa nhau hơn về tuyệt đối ngay cả khi tỷ lệ spacing/width không đổi). Layer cao nhất thường dành riêng cho power mesh vì cần dòng điện lớn và điện trở thấp nhất.

Trong naming convention TSMC, M1 là local layer, các layer "x" (standard-thickness) là intermediate routing, layer "y" (thicker) và "z" (even-thicker) là semi-global và global, và layer "u" (ultra-thick) là power distribution. Sự phân cấp này không tùy tiện: nó phản ánh yêu cầu điện của từng loại kết nối và khả năng sản xuất của process.

### 3.3 Preferred Routing Direction Giảm Coupling Giữa Adjacent Layer

Quy ước các layer kề nhau có hướng routing vuông góc (Horizontal ↔ Vertical xen kẽ) không chỉ là convention mà có cơ sở vật lý. Nếu M1 và M2 đều Horizontal, mọi wire dài trên M1 sẽ chạy song song với wire dài trên M2 với khoảng cách vertical cực nhỏ (bằng chiều dày dielectric giữa hai layer). Diện tích mặt đối diện lớn tạo ra capacitance dọc lớn giữa hai layer, gây ra coupling và crosstalk giữa hai layer. Khi các layer có hướng vuông góc, wire M1 horizontal và wire M2 vertical chỉ giao nhau tại điểm vuông góc, diện tích mặt đối diện gần bằng không, coupling giữa hai layer giảm đáng kể. Đây là DFM decision được embed vào technology từ đầu.

### 3.4 Via Là Điểm Kết Nối Đa Chiều Nhưng Cũng Là Nút Thắt RC Và Reliability

Via kết nối hai metal layer liền kề. Mỗi via có resistance đóng góp vào tổng RC của net. Via resistance của single-cut via tương đối cao so với wire của cùng chiều dài. Với multi-cut via có N cut song song, total resistance giảm theo R_single/N. Điều này tạo ra quan hệ trực tiếp: tăng số cut song song giảm via resistance, giảm delay, và cải thiện timing. Đồng thời, mỗi cut thêm vào chiếm diện tích, có thể gây congestion nếu space không đủ.

Via là điểm dễ fail nhất trong interconnect vì hai cơ chế. Cơ chế đầu là manufacturing defect: misalignment trong lithography làm via cut không tiếp xúc đầy đủ với metal layer trên hoặc dưới, gây tăng resistance hoặc open circuit. Cơ chế thứ hai là electromigration và thermal stress: dòng điện tập trung cao trong via cut nhỏ làm nguyên tử kim loại dịch chuyển theo thời gian, tạo void (lỗ hổng). Đây là lý do redundant via insertion là bước chip finishing quan trọng, và cũng là lý do foundry yêu cầu multi-cut via ở advanced node như một DRC requirement cứng.

---

## PHẦN IV: ROUTING TRACK, PITCH, VÀ ROUTING RESOURCE

### 4.1 Pitch Xác Định Routing Density Tối Đa Và Là Ranh Giới DRC

Pitch là khoảng cách giữa hai routing track kề nhau trên cùng layer, bằng tổng wire width và minimum spacing. Pitch nhỏ hơn cho phép nhét nhiều wire hơn vào cùng diện tích, tức là routing density cao hơn. Nhưng pitch nhỏ hơn cũng đồng nghĩa spacing giữa wire nhỏ hơn, coupling capacitance giữa adjacent wire lớn hơn, và nguy cơ DRC short trong fabrication cao hơn. Đây là trade-off căn bản nhất giữa density và reliability/SI.

Cell height được đo bằng số track pitch. Một 7-track cell có chiều cao bằng 7 lần M1 track pitch. Cell height ảnh hưởng trực tiếp đến routing resource bên trong cell vì cell hẹp hơn có ít track hơn cho internal routing. Cell hẹp → mật độ placement cao hơn → nhưng pin access khó hơn và internal routing tắc nghẽn hơn.

### 4.2 Routing Track Grid Là Không Gian Rời Rạc Mà Router Hoạt Động Trong Đó

Grid-based routing, phổ biến trong standard cell design, chỉ cho phép wire được đặt tại vị trí grid point. Mỗi routing track là một đường thẳng song song với preferred direction của layer đó, cách track kề nhau đúng một pitch. Router phải chọn từ tập hợp track rời rạc này, không thể đặt wire ở vị trí tùy ý. Điều này đơn giản hóa routing algorithm và đảm bảo DRC compliance tự động (vì mọi track đều đã ở spacing hợp lệ), nhưng đôi khi làm wire dài hơn cần thiết vì phải đi theo đường grid.

Gridless routing cho phép wire ở vị trí tự do không bị ràng buộc grid, phù hợp hơn cho analog hoặc custom layout nơi có nhiều wire width khác nhau và cần tối ưu từng trường hợp cụ thể.

---

## PHẦN V: GLOBAL ROUTING — QUÁ TRÌNH LẬP KẾ HOẠCH VÀ MỐI QUAN HỆ VỚI CONGESTION

### 5.1 GCELL Là Đơn Vị Trừu Tượng Hóa Routing Resource

Global routing chia chip thành lưới GCELL (Global Cell), mỗi GCELL là một vùng hình chữ nhật chứa nhiều routing track. Mỗi cạnh GCELL edge có routing capacity là số track có thể đi qua cạnh đó. Routing demand là số net thực sự cần đi qua cạnh đó. Overflow được định nghĩa là max(0, demand - capacity). Overflow lớn hơn 0 tại một GCELL edge có nghĩa là có nhiều net muốn đi qua hơn số track cho phép — đây là congestion hotspot.

Global routing không tạo wire thực. Nó chỉ lập kế hoạch: net A sẽ đi qua vùng GCELL X, Y, Z và được assign layer L. Kế hoạch này (routing guide) được chuyển cho detailed router để thực hiện. Chất lượng routing guide quyết định phần lớn chất lượng detailed routing outcome: nếu guide đưa nhiều net vào vùng congested, detailed router sẽ phải route detour dài hoặc fail routing một số net.

### 5.2 Congestion Là Nguyên Nhân Gốc Rễ Của Nhiều Vấn Đề Downstream

Congestion trong global routing tạo ra chuỗi vấn đề có liên quan. Đầu tiên, khi route demand vượt capacity, router phải chọn: một số net bị forced đi đường vòng (detour) qua vùng ít tắc nghẽn hơn. Detour dài hơn đồng nghĩa wire length tăng. Wire length tăng đồng nghĩa R và C tăng theo L². RC tăng đồng nghĩa delay tăng. Delay tăng gây setup violation trên các path đi qua những net bị detour.

Thứ hai, trong vùng congested cao, router có thể phải ép wire gần nhau hơn minimum spacing để fit. Điều này tạo DRC spacing violation, và cũng tăng coupling capacitance giữa adjacent wire (crosstalk worsens). Thứ ba, congestion nghiêm trọng có thể dẫn đến unrouted net — net không tìm được đường hợp lệ, tạo ra open circuit trong netlist.

Nguồn gốc của congestion thường là placement density quá cao ở một vùng, hoặc floorplan đặt các block cần kết nối nhiều nhưng ở xa nhau. Đây là lý do physical design engineer phải quan tâm congestion từ sớm (floorplan stage), không phải đợi đến routing mới xử lý.

### 5.3 Routing Blockage Kiểm Soát Congestion Và Bảo Vệ Vùng Nhạy Cảm

Hard blockage hoàn toàn cấm routing trong vùng được chỉ định. Router phải tìm đường vòng quanh hard blockage mà không có exception nào. Hard blockage được dùng bên dưới macro và IP block (router không được chui vào trong macro), trong vùng analog (digital routing noise sẽ interfere với analog signal), và trong các vùng DRC cứng. Bản chất của hard blockage là nó cắt giảm routing resource khả dụng cho region xung quanh, vì net phải đi vòng — điều này có thể làm tăng congestion ở vùng lân cận nếu blockage được đặt không cẩn thận.

Soft blockage ngăn cản routing nhưng cho phép nếu không còn lựa chọn. Router ưu tiên tránh soft blockage nhưng có thể dùng nó như last resort. Soft blockage hữu ích cho vùng không mong muốn có wire nhưng không phải cấm tuyệt đối, ví dụ vùng gần macro boundary nơi wire có thể gây noise nhưng không nhất thiết phải hoàn toàn cấm.

Mối quan hệ giữa routing blockage và DRC: blockage ngăn wire đi qua vùng mà nếu wire đi qua sẽ tạo DRC violation hoặc functional issue. Blockage là biện pháp phòng ngừa phía routing, DRC là biện pháp kiểm tra phía verification. Cả hai cùng nhau đảm bảo layout hợp lệ.

---

## PHẦN VI: ROUTING CONSTRAINTS VÀ NDR — KIỂM SOÁT CÓ MỤC TIÊU

### 6.1 NDR Là Biểu Hiện Vật Lý Của Yêu Cầu SI Và Reliability

Non-Default Rule (NDR) cho phép một net cụ thể được route với wire width hoặc spacing khác (thường lớn hơn) so với default rule. NDR không phải tùy chọn random mà có justification vật lý rõ ràng. Wire rộng hơn giảm điện trở R, cải thiện timing và EM margin. Spacing lớn hơn giảm coupling capacitance Cm với adjacent wire, giảm crosstalk. Cả hai cùng tiêu thụ nhiều routing resource hơn (mỗi net với NDR chiếm hai track thay vì một nếu double-width và double-spacing được áp dụng).

Clock net dùng NDR vì hai lý do: (1) clock phải distribute đều đến toàn bộ Flip-flop với skew nhỏ nhất có thể — wire rộng hơn giảm RC variation từ process variation; (2) clock switching liên tục là aggressor mạnh nhất trong chip — spacing lớn hơn ngăn clock coupling sang data net. Critical path dùng NDR để giảm delay. High-current net dùng NDR để tăng tiết diện, giảm current density, và giảm EM risk.

### 6.2 Shielding Là NDR Đặc Biệt Cho SI-Critical Net

Shielding là việc đặt wire nối GND (hoặc VDD) song song hai bên critical signal net. Wire shield có điện áp cố định, không switching, nên không inject noise vào signal. Đồng thời, shield "hấp thụ" coupling từ aggressors bên ngoài trước khi chúng tiếp cận victim. Same-layer shielding (GND — Signal — GND trên cùng layer) giảm lateral coupling. Adjacent-layer shielding (GND plane trên layer kề) giảm vertical coupling.

Shielding tiêu tốn routing resource gấp ba (GND wire + Signal wire + GND wire), hoặc gấp đôi nếu chỉ một bên. Đây là trade-off lớn về congestion và area. Shielding chỉ được áp dụng cho net thực sự critical: clock net, analog signal, và data path có setup margin cực kỳ nhỏ.

### 6.3 Priority-Based Routing Phản Ánh Thứ Tự Quan Trọng

Router xử lý net theo thứ tự ưu tiên. Net critical được route đầu tiên, chọn path ngắn nhất trên layer tốt nhất, ít detour. Net thông thường route sau, dùng routing resource còn lại. Net local route cuối, fill vào space còn trống. Thứ tự này đảm bảo critical net có routing quality tốt nhất, trong khi net ít quan trọng hơn chấp nhận routing quality thấp hơn. Nếu thứ tự bị đảo, critical net có thể bị forced đi đường dài vì space đã bị chiếm bởi net ít quan trọng hơn.

---

## PHẦN VII: DETAILED ROUTING — TỪ KẾ HOẠCH ĐẾN WIRE THỰC TẾ

### 7.1 Detailed Routing Là Quá Trình Chuyển Đổi Topology Thành Geometry

Detailed routing nhận routing guide từ global routing và tạo ra metal shape thực tế với tọa độ chính xác, layer cụ thể, via ở vị trí đúng. Đây là bước đầu tiên tạo ra wire có thể được kiểm tra bởi DRC và đo được RC bởi extraction tool. Sự chuyển đổi từ "kế hoạch GCELL" sang "wire thực tế" không phải lúc nào cũng suôn sẻ vì global routing có thể lạc quan về routing resource trong một GCELL — thực tế khi detailed router cố gắng đặt wire đúng track, có thể phát hiện rằng track đã bị chặn bởi blockage hoặc via của wire khác.

Rip-up and Reroute là cơ chế giải quyết conflict này: detailed router có thể tạm thời chấp nhận DRC violation, sau đó rip up (xóa) phần wire vi phạm và reroute theo hướng khác. Quá trình này iterative và có thể tốn nhiều thời gian trong vùng congested.

### 7.2 Pin Access Problem Là Giao Điểm Giữa Placement Và Routing

Pin access problem xảy ra khi router không thể tiếp cận pin của cell mà không vi phạm DRC. Nguyên nhân phổ biến nhất là hai cell đặt quá gần nhau, wire của cell này chặn pin của cell kia. Pin nằm trên M1 nhưng wire M2 của cell bên cạnh chạy qua vùng có thể route xuống M1 — không còn path hợp lệ.

Pin access problem là minh chứng rõ nhất cho quan hệ giữa placement và routing. Placement quality không chỉ ảnh hưởng timing thông qua wire length mà còn ảnh hưởng routing feasibility thông qua pin accessibility. Giải pháp bao gồm cell flipping (mirror cell theo trục ngang), tạo gap giữa cell, pin swapping (dùng pin tương đương nhưng accessible hơn), hoặc via stack (tiếp cận pin từ layer cao hơn qua nhiều via). Nếu không fix được, pin access failure dẫn đến unrouted net, tức là open circuit.

### 7.3 DRC Violation Sau Detailed Routing Cần Được Phân Loại Theo Mức Độ Nghiêm Trọng

Sau detailed routing, DRC violation được phân thành hai nhóm với xử lý khác nhau. Short và open là nhóm critical phải được giải quyết hoàn toàn trước khi tiếp tục bất kỳ bước nào. Lý do là RC extraction dựa trên connectivity của netlist. Nếu có short (hai net chạm nhau), extraction sẽ tính RC sai vì topology mạng đã thay đổi. STA sau đó nhận RC sai sẽ cho kết quả timing không đáng tin. Nếu có open, net bị đứt không có path hoàn chỉnh, STA không thể phân tích path đó, và chip đương nhiên sẽ fail về function.

Spacing, width, và enclosure violation thuộc nhóm có thể handle trong post-route optimization nếu số lượng ít và isolated. Nếu lan rộng (widespread), đây là dấu hiệu routing convergence problem bắt nguồn từ congestion, cần quay lại floorplan hoặc placement.

---

## PHẦN VIII: PARASITIC RC — TRỤC TRUNG TÂM CỦA TIMING ANALYSIS

### 8.1 Sự Chênh Lệch Giữa Estimated Và Actual RC Là Nguyên Nhân Cần Post-Route Optimization

Trước khi routing (trong Placement và CTS), STA tool dùng estimated parasitic RC. Estimation này dựa trên statistical model (ví dụ: wire dài trung bình bao nhiêu cho loại net này, bao nhiêu via thường có) mà không có wire thực. Sau routing, RC extraction tool tính actual RC từ exact geometry của từng wire và via đã được route. Hai bộ số này gần như luôn khác nhau, thường là actual RC lớn hơn vì: wire bị forced đi vòng do congestion nên dài hơn dự kiến; số via thực tế nhiều hơn ước tính do nhiều layer change; coupling capacitance từ wire kề nhau không được model đúng trong simple estimation.

Hậu quả trực tiếp: delay thực tế lớn hơn delay ước tính, slack thực tế thấp hơn. Điều này giải thích tại sao sau routing luôn có thêm violation mà pre-route không thấy. Post-route optimization là giai đoạn duy nhất có thể fix các violation này vì chỉ sau khi routing mới có RC thực tế để làm việc với.

### 8.2 SPEF Là File Chứa RC Thực Tế Và Là Input Bắt Buộc Cho STA Signoff

SPEF (Standard Parasitic Exchange Format) là output của RC extraction, chứa R và C của từng wire và via trong design. SPEF được tạo ra bởi extraction tool chuyên dụng (Cadence Quantus, Synopsys StarRC) sử dụng ITF file và layout geometry (từ GDS hoặc DEF). SPEF được tạo riêng cho từng process corner vì R và C của metal thay đổi theo process variation, nhiệt độ, và điện áp. STA signoff tool (PrimeTime, Tempus) đọc SPEF cùng với LIB và SDC để tính delay chính xác cho từng path.

Chuỗi phụ thuộc hoàn chỉnh: Foundry cung cấp ITF và LIB. Layout (DEF/GDS) được tạo bởi PnR. ITF + Layout → Extraction tool → SPEF. SPEF + LIB + SDC → STA tool → Timing report. Nếu bất kỳ mắt xích nào sai (ITF không đúng corner, layout có DRC violation, SDC thiếu constraint), STA result không phản ánh silicon thực.

### 8.3 OCV Là Bổ Sung Cho RC Để Phản Ánh Biến Đổi Trong Một Die

RC extraction cho kết quả nominal — giá trị "expected" tại một PVT corner. Nhưng trong thực tế, các transistor và wire trên cùng một die không hoàn toàn identical do process variation cục bộ. OCV (On-Chip Variation) modeling dùng derate factor để bù đắp: launch clock path và data path được derate theo hướng pessimistic (delay tăng thêm X%), capture clock path được derate theo hướng optimistic (delay giảm X%). Điều này tạo ra "worst-case" scenario thực tế hơn nominal.

CPPR (Clock Path Pessimism Removal) giải quyết vấn đề OCV bị apply hai lần cho common clock path. Khi launch và capture Flip-flop share một đoạn clock path chung, OCV sẽ derate đoạn đó một lần cho launch (pessimistic) và một lần cho capture (optimistic), nhưng vì đó là cùng một đoạn vật lý, không thể cùng lúc chậm hơn và nhanh hơn. CPPR loại bỏ phần pessimism bị double-counted này, cho ra kết quả timing thực tế hơn.

---

## PHẦN IX: CROSSTALK — TỪ VẬT LÝ ĐẾN TIMING VÀ FUNCTION

### 9.1 Coupling Capacitance Là Nguyên Nhân Vật Lý Của Mọi Crosstalk Effect

Khi hai wire chạy song song gần nhau, điện trường giữa chúng tạo ra coupling capacitance Cm. Đây là tụ điện ký sinh không mong muốn nhưng không thể tránh khỏi trong mọi multi-layer metal stack. Giá trị Cm tỷ lệ thuận với chiều dài song song và tỷ lệ nghịch với khoảng cách: Cm ∝ L_parallel / d. Tỷ lệ Cm/(Cm + C_ground) là "coupling ratio" — tỷ lệ này quyết định mức độ ảnh hưởng của aggressor lên victim. Coupling ratio cao hơn đồng nghĩa aggressor có thể drive victim dễ dàng hơn.

Khi aggressor switching với dV/dt nhất định, coupling current I = Cm × dV/dt chạy qua Cm sang victim net. Nếu victim đang transitioning, current này làm tăng tốc (same direction) hoặc làm chậm (opposite direction) transition của victim. Nếu victim đang stable, current này tạo glitch (voltage transient) trên victim net.

### 9.2 Ba Loại Ảnh Hưởng Crosstalk Tương Ứng Ba Loại Risk Khác Nhau

Crosstalk speed-up (negative crosstalk theo định nghĩa học thuật) xảy ra khi aggressor và victim chuyển cùng chiều. Coupling current bổ sung vào dòng của victim driver, làm victim chuyển nhanh hơn. Hậu quả timing: data arrive sớm hơn dự kiến tại capture Flip-flop, tức là data path delay giảm. Nếu hold time requirement không được met (minimum data path delay quá nhỏ), hold violation xảy ra. Crosstalk speed-up là nguyên nhân tiềm ẩn của hold violation mà không phải lúc nào cũng rõ ràng.

Crosstalk slow-down (positive crosstalk) xảy ra khi aggressor và victim chuyển ngược chiều. Coupling current cản trở victim transition do hiệu ứng Miller: capacitance effective tăng lên gấp đôi (từ quan điểm của victim driver, Cm trông như 2×Cm vì voltage của aggressor di chuyển ngược chiều so với victim). Data path delay tăng, setup timing bị tác động tiêu cực, setup violation có thể xuất hiện.

Glitch (crosstalk noise) xảy ra khi aggressor switching nhưng victim đang stable. Voltage spike trên victim net có thể: (1) vượt ngưỡng logic threshold và được capture bởi Flip-flop như data hợp lệ sai → functional error; (2) không đủ vượt threshold nhưng làm victim chuyển state tạm thời, gây race condition ở node kề; (3) gây tăng dynamic power vì switching không mong muốn.

### 9.3 Crosstalk Analysis Phụ Thuộc Vào Sự Hoàn Chỉnh Của Routing Và RC Extraction

Crosstalk analysis chỉ chính xác sau khi có actual wire geometry. Pre-route crosstalk analysis chỉ là approximation dựa trên placement proximity. Sau routing, RC extraction tính toán Cm cho từng cặp wire kề nhau dựa trên layout thực tế. Giá trị Cm này được include trong SPEF và được SI-aware STA tool sử dụng để tính coupling-adjusted delay cho từng path. Đây là lý do SI-driven timing được enable muộn trong post-route loop, không phải trong initial setup.

Trong practice (được xác nhận bởi nguồn từ Synopsys và Cadence), STA với SI enabled tốn thời gian tính toán lớn hơn nhiều so với STA không có SI vì phải xét tất cả các tổ hợp aggressor-victim cho mỗi net. Đây là trade-off giữa accuracy và runtime.

### 9.4 Biện Pháp Giảm Crosstalk Trong Routing Và Quan Hệ Với Các Concept Khác

Wire spacing NDR là biện pháp đầu tiên và phổ biến nhất. Spacing lớn hơn giảm Cm theo tỷ lệ nghịch khoảng cách, trực tiếp giảm coupling. Cost: tiêu thụ thêm routing resource, tăng congestion nếu áp dụng rộng rãi.

Shielding (đã đề cập ở mục routing constraints) là biện pháp mạnh nhất nhưng tốn kém nhất về resource.

Net ordering (sắp xếp thứ tự net trong track) là biện pháp free về resource: đảm bảo victim không nằm cạnh worst aggressor mà không cần thêm wire hay track. Tool thực hiện điều này bằng cách re-order routing trong GCELL.

Re-buffering aggressor giảm switching strength của aggressor, giảm dV/dt, giảm coupling current I = Cm × dV/dt. Upsizing victim driver tăng drive strength của victim, giảm output impedance, làm victim ít bị ảnh hưởng bởi coupling noise (low-impedance driver dập tắt noise nhanh hơn).

Routing trên layer khác (layer với preferred direction vuông góc) giảm parallel run length giữa aggressor và victim, giảm tổng Cm. Đây là biện pháp radical nhất và chỉ dùng khi các biện pháp khác không đủ.

---

## PHẦN X: TIMING — TOÀN BỘ HỆ THỐNG CHECK VÀ MỐI QUAN HỆ

### 10.1 Setup Và Hold Là Hai Ràng Buộc Đối Ngịch Trên Cùng Data Path

Setup violation và hold violation là hai phía của cùng một timing constraint nhưng fix theo hướng ngược nhau. Setup violation xảy ra khi data path quá chậm — data đến sau clock edge trừ setup time của capture Flip-flop. Fix: giảm delay bằng cách upsize cell, insert buffer vào điểm thắt cổ chai, reroute wire critical để ngắn hơn, hoặc delay capture clock (useful skew). Hold violation xảy ra khi data path quá nhanh — data thay đổi sớm hơn capture clock edge cộng hold time. Fix: tăng delay bằng cách downsize cell (cell nhỏ hơn có delay lớn hơn), insert buffer delay, hoặc advance launch clock (useful skew ngược chiều).

Vấn đề là fix setup có thể worsen hold và ngược lại. Upsize cell giảm delay (tốt cho setup) nhưng có thể làm hold tệ hơn nếu path đó gần hold limit. Đây là lý do post-route optimization phải xét cả setup và hold đồng thời và có thể cần iterate nhiều lần.

### 10.2 DRV Là Timing Violation Không Phải DRC Violation Hình Học

DRV (Design Rule Violation về timing, không phải layout DRC) bao gồm transition violation và capacitance violation. Transition violation: slew của một net vượt max_transition quy định trong LIB. Nguyên nhân: driver quá yếu so với total load capacitance của net. Khi slew quá lớn (tín hiệu chuyển chậm), cell ở downstream "nhìn thấy" input transition chậm, điều này thêm vào cell delay của downstream cell và có thể làm fail setup hoặc làm fail hold (nếu slew chậm làm fast path trở nên chậm hơn yêu cầu hold). Capacitance violation: total load capacitance của net vượt max_capacitance của driver. Nguyên nhân: quá nhiều fanout hoặc wire quá dài. Fix: insert buffer để split net thành hai đoạn ngắn hơn, mỗi đoạn có load nhỏ hơn.

### 10.3 Slack, WNS, TNS Là Metrics Phản Ánh Health Của Toàn Bộ Design

Slack của một path là T_required - T_arrival. Positive slack: path đáp ứng timing, có margin dự phòng. Negative slack: path fail timing. WNS (Worst Negative Slack) là slack âm tệ nhất trong toàn design — phản ánh mức độ nghiêm trọng của violation đơn lẻ tệ nhất. TNS (Total Negative Slack) là tổng cộng tất cả slack âm — phản ánh tổng "công việc" phải làm để fix timing.

WNS và TNS cùng nhau cho bức tranh toàn cảnh. WNS tệt nhưng TNS nhỏ có nghĩa là một path rất tệ nhưng ít path vi phạm — có thể fix được. WNS trung bình nhưng TNS rất lớn có nghĩa là nhiều path đều vi phạm một chút — cần fix hệ thống. Trong post-route optimization, engineer theo dõi cả hai metric sau mỗi iteration.

### 10.4 Timing Path Groups Cho Phép Targeted Optimization

Bốn nhóm path chính in2reg (input port đến Flip-flop), reg2reg (Flip-flop đến Flip-flop), reg2out (Flip-flop đến output port), in2out (input đến output thẳng) có đặc điểm timing khác nhau. reg2reg là nhóm lớn nhất và quan trọng nhất vì đây là nơi hầu hết logic nằm. in2reg bị ràng buộc bởi input delay constraint trong SDC. reg2out bị ràng buộc bởi output delay constraint. ICG path group (in2icg, reg2icg) là nhóm đặc biệt cho Integrated Clock Gate — enable signal của ICG phải arrive trước clock edge để gating chính xác, nếu enable đến trễ, clock bị cắt sai và gây functional failure không phải timing failure.

Phân nhóm cho phép tool biết đang optimize nhóm nào và tránh overfit một nhóm trong khi bỏ qua nhóm khác. Phân nhóm cũng làm timing report dễ debug hơn vì violation được nhóm theo loại path.

### 10.5 Full Clock Path Expansion Là Bắt Buộc Để Debug Timing Violation Đúng

Khi analyze timing violation, chỉ xem data path mà không xem clock path là không đủ. Setup slack phụ thuộc vào T_capture_clock_arrival + T_period - T_setup - T_launch_clock_arrival - T_data. Hold slack phụ thuộc vào T_launch_clock_arrival + T_data - T_capture_clock_arrival - T_hold. Nếu clock skew lớn (T_capture - T_launch lớn), setup và hold margin đều thay đổi. Full clock path expansion hiển thị cả launch clock path và capture clock path, cho phép engineer nhận biết ngay là violation do data path hay do clock path không cân bằng. Fix sai vì thiếu thông tin clock là nguyên nhân phổ biến của timing regression.

---

## PHẦN XI: POST-ROUTE OPTIMIZATION — TINH CHỈNH TRÊN NỀN TẢNG RC THỰC TẾ

### 11.1 Post-Route Optimization Là Tầng Cuối Cùng Của Implementation

Post-route optimization là giai đoạn làm việc với actual RC. Mọi transformation trước đó đều dùng estimated RC. Đây là tầng duy nhất biết chính xác cost vật lý của mỗi wire và via. Nguyên tắc quan trọng nhất: thay đổi phải localized và minimal-impact. Lý do là detailed routing đã tốn nhiều thời gian để tối ưu wire và via globally. Mọi thay đổi placement hoặc routing đều có nguy cơ phá vỡ optimization đó và introduce regression (thay đổi RC trên net không vi phạm, gây violation mới).

Có ba loại transformation phổ biến. Selective re-buffering: insert buffer tại điểm giữa wire dài để giảm effective RC (buffer chia wire thành hai đoạn ngắn hơn, mỗi đoạn có RC nhỏ hơn). Cell resizing: upsize/downsize cell để tăng/giảm delay. Critical net re-routing: route lại net trên layer có RC thấp hơn hoặc chọn path ngắn hơn.

### 11.2 Cell Resizing Là Trade-off Giữa Drive Strength, Delay, Power, Và Area

Drive strength của cell tỷ lệ với W/L của transistor. Upsize (tăng W) tăng drive current, giảm output RC delay, cải thiện slew — giúp setup và transition violation. Nhưng transistor rộng hơn chiếm area lớn hơn và leakage power tỷ lệ với W cao hơn. Khi upsize một cell, không gian vật lý cần thiết tăng — nếu neighboring cell không có khoảng cách đủ, tool phải legalize lại (di chuyển cell lân cận) và điều này có thể làm thay đổi RC của wire kết nối với các cell bị di chuyển, tạo cascade effect.

Downsize (giảm W) tăng output resistance, tăng cell delay — giúp hold violation. Nhưng downsize quá nhiều có thể làm slew xấu đi và tạo transition violation trên net đó. Engineer phải cân bằng giữa hold fix và transition DRV risk. Corner-aware sizing thêm một chiều phức tạp: cell cần đủ drive strength ở slow corner (SS, high temperature) để pass setup, nhưng không quá lớn ở fast corner (FF, low temperature) để không worsen hold.

### 11.3 Useful Skew Là Công Cụ Powerful Nhưng Cần Thận Trọng

Useful skew điều chỉnh clock arrival time tại Flip-flop để create asymmetric timing margin: delay clock tại capture FF tăng setup window cho path đó (capture clock đến muộn hơn cho data thêm thời gian), advance clock tại launch FF tăng hold margin (launch clock đến sớm hơn, data cũng đến sớm hơn so với hold boundary). Useful skew không thay đổi RC của data path — nó thay đổi T_required một cách hợp lệ.

Nguy hiểm của useful skew: mỗi Flip-flop thường là điểm launch cho nhiều path và điểm capture cho nhiều path khác. Delay clock tại FF_X giúp path A (vào FF_X) nhưng hurt path B (ra từ FF_X) vì bây giờ launch clock của path B đến trễ hơn, giảm setup margin của path B. Useful skew cần được optimize globally, không thể áp dụng localized mà không xét tác động lên tất cả path liên quan.

---

## PHẦN XII: ECO FLOW — THAY ĐỔI CÓ KIỂM SOÁT MUỘN TRONG FLOW

### 12.1 ECO Là Giải Pháp Cho Dilemma Giữa Fix Quality Và Flow Stability

Dilemma căn bản: sau routing, running full place-and-route lại có thể cho kết quả tốt hơn nhưng mất ngày đến tuần. ECO fix violation bằng minimum change nhưng kết quả có thể không optimal. Industry chấp nhận ECO vì time-to-market quan trọng hơn absolute optimal quality.

Timing ECO và functional ECO là hai loại phục vụ mục đích khác nhau. Timing ECO chỉ thay đổi cell size/drive strength/placement cục bộ mà không alter logic function. Functional ECO thay đổi logic gate (fix bug phát hiện muộn) và cần LEC verify sau đó. L12 chỉ đề cập timing ECO.

### 12.2 ECO Flow Là Vòng Lặp Tối Thiểu Hóa Tác Động Xây Dựng Trên Convergence

ECO flow là vòng lặp chặt chẽ giữa STA tool và PnR tool. STA tool phân tích violation, generate ECO command file (chỉ định cell cụ thể cần resize/swap/insert/delete, net cần add, pin cần connect). PnR tool thực thi ECO command với eco_mode = true — trong mode này, tool chỉ legalize cell bị ECO và chỉ reroute net bị affect, không re-optimize globally. Sau khi ECO thực thi, re-run STA để verify fix không introduce violation mới. Iterate cho đến khi clean.

Nguyên tắc ECO routing: chỉ route lại net bị modify, giữ nguyên routing của net không liên quan. Incremental routing thay vì full reroute. Legalize chỉ cell bị affect. Lý do: bất kỳ wire nào bị touched đều có RC thay đổi, potential timing regression. Minimum disruption = minimum regression risk.

Nguồn từ Synopsys ECO whitepaper xác nhận: "signoff-accurate timing and extraction tools for ECO guidance become increasingly important" — ECO command phải được generate từ signoff STA (PrimeTime), không phải in-tool STA, để đảm bảo ECO fix đúng violation và không bị timing discrepancy giữa engine.

---

## PHẦN XIII: POWER VÀ AREA RECOVERY — TẬN DỤNG SLACK DƯ THỪA

### 13.1 Positive Slack Là "Vốn" Để Recovery

Trong các giai đoạn optimization trước (synthesis, CTS, pre-route optimization), tool thường aggressive để đảm bảo timing closure — upsize cell, insert buffer, dùng LVT cell. Kết quả là sau timing closure, nhiều path có positive slack lớn hơn cần thiết. Path này đang "over-engineered" — tốn power và area mà không có benefit thêm.

Recovery không phải là "lấy lại" gì đã mất. Đây là bước tối ưu PPA trong khi maintain timing. Điều kiện bắt buộc: mọi thay đổi recovery phải không làm slack xuống dưới 0. Nếu downsize một cell trên path có 200ps slack, slack có thể giảm xuống 150ps — vẫn positive, vẫn valid. Tool luôn check timing sau mỗi recovery change.

### 13.2 VT Swap Khai Thác Quan Hệ Exponential Giữa Threshold Voltage Và Leakage

Leakage current tỷ lệ với exp(-Vt/(nkT/q)). Vt tăng thêm 100mV có thể giảm leakage tới 10 lần (tùy process và temperature). Đây là lever mạnh nhất cho power reduction. Cell HVT chậm hơn LVT (delay lớn hơn khoảng 20-30% tùy process) nhưng leakage thấp hơn nhiều bậc. VT swap từ LVT sang HVT trên non-critical path tận dụng positive slack dư để đổi lấy power saving lớn.

Foundry thường cung cấp standard cell library ở nhiều Vt flavor (ULVT, LVT, SVT, HVT). Design flow thường bắt đầu với nhiều LVT cell để đảm bảo timing, sau đó recovery phase swap về HVT ở những chỗ có thể.

### 13.3 Cell Downsizing Tác Động Đến Cả Power, Area, Và Leakage

Cell nhỏ hơn (drive strength nhỏ hơn) tiêu thụ ít dynamic power hơn (switching power ∝ C×V²×f, C tỷ lệ với kích thước transistor), leakage nhỏ hơn (transistor narrow hơn), và physical footprint nhỏ hơn (area giảm). Ba benefit cùng lúc. Trade-off duy nhất là delay tăng — phải có đủ positive slack.

Redundant buffer và inverter pair là "rác" từ quá trình optimization. Buffer được insert ở stage trước để fix timing có thể trở thành over-buffered sau khi ECO thêm buffer ở chỗ khác. Inverter pair (hai inverter liên tiếp) không làm thay đổi logic function nhưng thêm area và power. Recovery xóa những cell này khi slack đủ.

---

## PHẦN XIV: CHIP FINISHING VÀ DFM — ĐẢM BẢO KHẢ NĂNG SẢN XUẤT

### 14.1 DFM Là Tập Hợp Quyết Định Physical Giảm Thiểu Fabrication Risk

DFM (Design For Manufacturability) không phải là một kỹ thuật đơn lẻ mà là triết lý xuyên suốt: thiết kế không chỉ cần đúng về điện mà còn phải robust với process variation trong fabrication. Mỗi bước DFM trong chip finishing giải quyết một loại fabrication failure mode cụ thể.

### 14.2 Filler Cell Giải Quyết Vấn Đề Discontinuity Trong Cell Row

Sau routing, empty site (vị trí trống không có cell) xuất hiện trong cell row. Empty site làm N-well layer và implant layer bị đứt đoạn. Trong CMOS process, N-well phải continuous để well tie point (kết nối N-well với VDD) hoạt động đúng — nếu đứt, một phần N-well floating và có thể trigger latchup. Implant layer (p+ và n+ dopant) cũng cần continuous để đảm bảo threshold voltage uniform của transistor trong cùng cell row.

Filler cell không có logic function nhưng có N-well và implant giống cell thông thường, lấp đầy empty site để đảm bảo continuity. Power rail VDD và VSS cũng cần continuous để tất cả cell trong row nhận nguồn — filler cell duy trì power rail qua chỗ trống. Đây là lý do filler cell insert là DRC requirement, không phải DFM optional.

### 14.3 Decap Cell Giải Quyết Dynamic IR Drop Bằng Nguyên Lý Local Charge Reservoir

Dynamic IR drop không phải là resistance problem mà là inductance/capacitance problem. Khi nhiều cell switching đồng thời, peak current demand vượt quá khả năng cung cấp ngay lập tức của power mesh (vì power mesh có inductance L, điện áp cần thời gian để settle). Decap cell là tụ điện lớn (thường hundreds of femtoFarads đến picoFarads) nằm giữa VDD và VSS, được đặt gần cell hay switching. Khi VDD sụt, decap xả điện tích ngay lập tức bù vào, giữ VDD ổn định đủ lâu cho dòng từ power mesh kịp đến.

Mối quan hệ với filler cell: PnR tool insert decap cell vào empty site trước tiên (để maximize decap coverage), sau đó mới dùng non-cap filler cho phần còn lại. Decap và non-cap filler cùng giải quyết empty site problem nhưng decap có benefit thêm về power integrity.

### 14.4 Redundant Via Giải Quyết Via Reliability Bằng Redundancy

Đã đề cập ở Phần IV về via geometry. Tại chip finishing stage, redundant via insertion là systematic pass: tool scan toàn bộ layout, xác định mọi single-cut via, và cố gắng swap sang multi-cut via ở những chỗ có đủ space. Via liability metric prioritize net có nhiều single-cut via nhất và current cao nhất — những net này có EM risk cao nhất nếu via fail.

Quan hệ với timing: via song song có R_total = R_single/N. Nếu swap từ 1-cut sang 2-cut, via resistance giảm một nửa. Điều này giảm RC nhẹ và có thể cải thiện timing một chút trên net đó. Nhưng tác động timing là secondary — primary goal là reliability.

### 14.5 Wire Spreading Và Widening Giải Quyết Lithography Yield Risk

Sau routing, nhiều wire nằm ở minimum spacing (chỉ đủ để pass DRC, không có margin cho lithography variation). Pattern bridging (short) có thể xảy ra nếu lithography in hai wire gần nhau thành một. Wire spreading tăng spacing vượt minimum ở những chỗ có space — giảm bridge risk. Wire widening tăng width của non-critical wire — giảm line-edge roughness effect và tăng EM margin.

Hai kỹ thuật này không áp dụng cho critical timing net vì thay đổi RC sẽ ảnh hưởng timing đã close. Đây là sự đối nghịch thú vị: timing net cần wire optimal nhỏ để tối thiểu C, nhưng non-timing net thì widening giúp reliability mà không ảnh hưởng gì đến function.

### 14.6 Metal Fill Giải Quyết CMP Non-Uniformity — Nhưng Tạo Timing Risk Mới

CMP (Chemical Mechanical Polishing) là bước làm phẳng bề mặt giữa các metal layer trong fabrication. CMP rate phụ thuộc vào density — vùng metal thưa bị mài mòn nhiều hơn (dielectric loss và erosion), vùng metal dày bị mài không đều (dishing). Hệ quả: wire resistance của net đi qua vùng bị dishing lớn hơn thiết kế, vì tiết diện thực tế sau CMP nhỏ hơn nominal. Timing prediction sau extraction dùng nominal geometry — không chính xác với silicon thực.

Metal fill giải quyết bằng cách thêm dummy metal shape trong vùng thưa để equalize density. Floating fill (không kết nối) đơn giản nhưng tạo coupling capacitance với wire kề. Grounded fill (kết nối GND) vừa equalize density vừa hoạt động như shield giảm coupling. Timing-aware fill dùng cost model để tránh đặt fill gần critical net — fill gần clock net hay critical data net có thể thêm đủ capacitance để flip timing từ pass sang fail.

Staggered fill pattern (fill trên layer liền kề được offset) giảm cross-layer coupling giữa fill shapes — vì fill không thẳng hàng theo chiều Z, coupling capacitance vertical giữa fill trên M2 và M3 nhỏ hơn so với non-staggered.

---

## PHẦN XV: POST-ROUTE SIGNOFF — HỆ THỐNG VERIFICATION ĐA TẦNG

### 15.1 Signoff Là Sự Hội Tụ Của Tất Cả Verification Domain

Post-route signoff không phải một check duy nhất mà là tập hợp verification cover ba domain riêng biệt: physical (DRC, LVS, Antenna, ERC), logical (LEC), và performance (STA, gate simulation, EM/IR). Ba domain này độc lập nhau về cơ chế nhưng có quan hệ qua layout: layout là "ground truth" chung. Physical verification dùng layout geometry. LEC dùng netlist derived từ layout. STA dùng RC derived từ layout. Nếu layout có lỗi, tất cả domain đều bị ảnh hưởng theo cách của từng domain.

### 15.2 DRC Là Rào Cản Đầu Tiên — Layout Hợp Lệ Về Hình Học

DRC verify rằng mọi metal shape trong layout đều tuân thủ foundry manufacturing rule. Rule phát sinh từ thực tế fabrication: lithography có giới hạn độ phân giải, plasma etch có variation, overlay có error. Rule là boundary đảm bảo design survive các variation này với yield đủ cao.

DRC violation là symptom có thể có nhiều nguyên nhân: router fail DRC trong congested area, ECO tạo DRC khi insert/resize cell, metal fill tạo spacing violation với existing wire, filler cell tạo issue với adjacent cell. Không phải mọi DRC violation đều sinh ra trong routing — chip finishing bước nào cũng có thể introduce DRC mới, và đây là lý do DRC phải chạy lại sau mỗi step.

In-tool DRC (trong PnR) và signoff DRC (Calibre, IC Validator) có accuracy khác nhau. In-tool DRC nhanh nhưng có thể miss một số rule phức tạp hoặc false positive. Signoff DRC dùng rule deck chính thức từ foundry, là standard duy nhất foundry chấp nhận.

### 15.3 LVS Là Rào Cản Thứ Hai — Layout Khớp Với Netlist

LVS (Layout vs. Schematic) verify connectivity: tool extract netlist từ layout geometry (metal connectivity, via, device) và compare với post-route netlist từ PnR. Short trong layout (hai metal touch vô tình) → LVS thấy extra connection → fail. Open trong layout → LVS thấy thiếu connection → fail. Missing device → LVS thấy cell trong netlist không có trong layout → fail.

Mối quan hệ LVS và RC extraction quan trọng: nếu LVS fail vì short, extraction sẽ thấy short đó và tính RC của net bị short cùng nhau — RC này sai, STA sẽ cho kết quả sai. LVS clean là prerequisite cho trustworthy extraction và trustworthy STA.

### 15.4 Antenna Check Kết Nối Fabrication Process Với Layout Geometry

Antenna check có lý do vật lý phức tạp hơn DRC spacing rule. Trong plasma etch, kim loại dài tích điện vì plasma ion bombardment. Điện tích tích lũy tạo điện áp. Nếu điện áp đủ lớn và được discharge qua gate oxide của transistor thay vì qua connection với source/drain, oxide bị stress và có thể broken down (Fowler-Nordheim tunneling hoặc hot carrier injection). Antenna ratio = diện tích metal tích điện / diện tích gate oxide kết nối. Foundry set maximum antenna ratio cho từng layer.

Antenna diode là fix: insert cell có diode giữa net và GND ngay tại pin bị vi phạm. Diode discharge điện tích xuống GND trong khi layer đang được etch, trước khi đủ để phá oxide. Via jumper là fix thứ hai: route wire vi phạm lên layer cao hơn (etch sau, ít tích điện hơn).

Antenna check trong PnR (trong routing step với `route_detail_fix_antenna true`) là first pass fix. Signoff antenna check là final verification với rule deck đầy đủ từ foundry.

### 15.5 ERC Bắt Electrical Violation DRC Và LVS Bỏ Sót

ERC check floating input (input không kết nối): gate transistor floating → trạng thái không xác định → có thể draw continuous current qua CMOS pull-up/pull-down stack → chip overheating. Multiple driver check (hai output drive same net): two drivers cùng drive ngược chiều → short circuit path từ VDD đến VSS qua cả hai driver → potentially destructive.

ESD check verify rằng mỗi IO pad có ESD protection structure. Khi điện tích tĩnh discharge qua IO pin (từ người hay machine), ESD structure clamp điện áp và shunt dòng xuống GND trước khi reaching internal circuit. Nếu không có ESD protection đủ mạnh, gate oxide của transistor input sẽ bị breakdown ngay lập tức.

Latchup check verify tap cell spacing. Tap cell cung cấp well contact (N-well kết nối VDD, P-substrate kết nối GND) để giữ well potential ổn định. Nếu tap cell quá thưa (spacing vượt foundry limit), phần well giữa hai tap không được "anchored" đủ — parasitic PNPN thyristor (gồm PMOS body và NMOS body kết hợp) có thể được trigger bởi noise hoặc injection current, dẫn đến latchup (thyristor latched trong trạng thái conducting, short VDD-VSS, dòng không giới hạn, chip bị destroy hoặc cần power cycle).

Quan hệ giữa ERC và floorplanning: tap cell spacing, guard ring placement, và ESD cell placement thường được set trong floorplan stage. Nếu floorplan sai, ERC fail rất khó fix late in flow vì cần thay đổi placement rộng rãi.

### 15.6 LEC Là Formal Proof Rằng Implementation Không Thay Đổi Logic

LEC (Logical Equivalence Check) dùng mathematical proof (Boolean equivalence checking, SAT/BDD) thay vì simulation để verify hai netlist có cùng function. Không thể simulation đủ vector để cover tất cả states của design nhiều triệu gate. Formal proof là giải pháp duy nhất cho coverage đầy đủ.

LEC được chạy ở ba thời điểm: sau synthesis (RTL vs gate netlist), sau scan insertion (pre vs post scan), và sau PnR (pre-route vs post-route). Mỗi điểm check rằng transformation tương ứng không làm sai logic. ECO trong post-route timing closure có thể vô tình alter logic nếu engineer nhầm cell ID hoặc net connection — LEC sẽ catch điều này ngay.

LEC không phải luôn pass/fail binary. Có thể có exception (waiver) với justification. Ví dụ: Flip-flop được replace bằng scan Flip-flop sau scan insertion — về mặt kỹ thuật, function thay đổi (thêm scan path), nhưng functional mode vẫn equivalent. Tool cần được inform về những transformation có chủ đích này qua don't-care hoặc exception specification.

### 15.7 STA Signoff Là Tầng Timing Verification Chính Xác Nhất

STA signoff khác in-tool STA ở hai điểm chính. Thứ nhất, RC extraction của signoff tool (StarRC, Quantus) dùng full-field solver chính xác hơn nhiều so với in-tool extraction đơn giản. SPEF từ signoff extraction là "single source of truth" cho timing. Thứ hai, signoff STA cover đầy đủ tất cả PVT corners (5 đến 10 corners tùy foundry), không phải một hoặc hai corners như in-tool STA.

SDF file được generate từ STA signoff là output quan trọng. SDF chứa annotated delay cho từng cell và net, per corner. SDF được dùng để back-annotate delay vào gate-level simulation — nếu không có SDF, simulation không thể reflect actual timing sau fabrication và dynamic timing bug có thể không bị phát hiện.

Mối quan hệ STA signoff và EM/IR: IR drop làm VDD cell thấp hơn nominal, delay tăng, timing bị ảnh hưởng. Lý tưởng, STA signoff nên dùng VDD profile sau IR drop analysis (không phải nominal VDD) để cho kết quả chính xác. Đây là "EMIR-aware STA" — được một số team implement ở advanced node. Theo nguồn từ Cadence (EMSpice 2.1 research), EM-induced IR drop tăng thêm delay và làm timing signoff không đủ conservative nếu không xét EM.

### 15.8 EM Và IR Drop Có Mối Quan Hệ Hai Chiều Làm Trầm Trọng Lẫn Nhau

EM (electromigration) tạo void trong wire theo thời gian → wire resistance tăng → IR drop tăng (V_drop = I × R, R tăng). IR drop cao → VDD thấp → cell switching chậm hơn → peak current demand tăng hơn (cần thêm thời gian để charge load capacitance với current thấp hơn) → IR drop worsens further. Đây là positive feedback loop không tốt: EM worsens IR drop, IR drop tăng stress trên wire và via, worsens EM in return.

Static IR drop là steady-state và liên quan đến average current. Dynamic IR drop là transient, liên quan đến peak current khi nhiều cell switching. Dynamic IR drop lớn hơn static nhiều lần vì peak current có thể gấp nhiều lần average current trong một clock cycle. Switching activity file (VCD từ simulation hoặc SAIF từ estimation) là input bắt buộc cho dynamic IR analysis — không có switching activity, không tính được peak demand, không tính được dynamic drop.

Fix EM violation trên wire: widen wire (tăng tiết diện, giảm current density J=I/A). Fix EM violation trên via: thêm via song song (giảm current per via). Fix dynamic IR drop: insert decap cell gần hot spot (local charge buffer). Fix static IR drop: widen power strap hoặc add power strap trong vùng drop cao.

### 15.9 Gate Simulation Là Dynamic Complement Của Static STA

Gate-level simulation với SDF back-annotation simulate actual behavior của design dưới timing thực tế. STA là static — nó không thể phát hiện glitch (signal chuyển trạng thái nhiều lần trong một cycle do race condition giữa hai path converging tại same node), X-propagation (unknown state lan ra khi reset không đúng sequence), hay reset sequencing issue (release reset sai thứ tự giữa clock domains).

Gate simulation không phải full functional verification (không đủ vector để cover all states) mà là targeted check trên selected scenarios: reset sequence, initialization, critical operating mode. Nó complement STA bằng cách catch dynamic anomaly mà static model không capture.

---

## PHẦN XVI: CHUỖI NHÂN QUẢ TOÀN CỤC — KẾT NỐI TẤT CẢ KHÁI NIỆM

### 16.1 Từ Fabrication Process Đến Timing Violation

Fabrication process (technology node, nguyên vật liệu, chemistry) quyết định physical properties của metal stack (thickness, resistivity, permittivity của dielectric). Physical properties được mô tả trong ITF. ITF là input cho RC extraction. RC extraction từ actual wire geometry cho ra SPEF. SPEF là input cho STA. STA với SDC constraint tính slack cho mọi timing path. Slack âm là timing violation. Đây là chuỗi đầy đủ từ vật lý đến timing report.

### 16.2 Từ Placement Quality Đến Signoff Result

Placement density và topology quyết định global routing congestion. Congestion quyết định routing detour length. Detour length quyết định wire length. Wire length quyết định RC. RC quyết định delay. Delay quyết định slack. Slack quyết định cần bao nhiêu post-route optimization. Optimization nhiều đến đâu quyết định power và area recovery range. Power và area recovery quyết định final PPA. Final PPA cùng với DRC, LVS, LEC, EM/IR quyết định signoff result. Signoff result quyết định tape-out. Chuỗi này minh chứng tại sao floorplan quality tác động đến tape-out schedule.

### 16.3 Từ Coupling Capacitance Đến Functional Failure

Wire parallel length và spacing quyết định coupling capacitance Cm. Cm quyết định magnitude của crosstalk noise và crosstalk delay. Crosstalk delay quyết định timing closure difficulty (SI-induced setup/hold violation). Crosstalk noise magnitude so sánh với noise margin quyết định functional failure risk. Noise margin đến từ transistor threshold voltage và operating voltage. Technology scaling giảm spacing (tăng Cm), giảm operating voltage (thu hẹp noise margin), và tăng frequency (tăng dV/dt trên aggressor) — ba yếu tố cùng làm crosstalk worse ở advanced node. Đây là lý do SI analysis ngày càng quan trọng ở sub-14nm.

### 16.4 Từ Metal Density Đến Timing Accuracy

Metal density distribution sau routing quyết định CMP uniformity. CMP uniformity quyết định actual wire geometry sau fabrication (dishing, erosion). Actual geometry quyết định actual RC. Actual RC khác với designed RC → timing prediction từ extraction không hoàn toàn chính xác. Metal fill equalize density, cải thiện CMP uniformity, giảm sai lệch giữa designed và actual RC, làm timing signoff đáng tin cậy hơn. Đây là chuỗi từ DFM decision đến STA accuracy.

### 16.5 Từ Via Failure Đến Long-Term Reliability

Via single-cut có nguy cơ fail do manufacturing defect và EM. Via fail → net resistance đột biến → delay thay đổi đột ngột → timing fail in-field. Redundant via giảm nguy cơ bằng redundancy. Wire widening và spreading giảm nguy cơ EM trên wire. Decap cell giảm dynamic IR drop, giảm VDD stress trên transistor. Tap cell đủ dày giảm latchup risk. ESD protection đủ mạnh prevent gate oxide breakdown. Tất cả cùng nhau tạo thành reliability margin — chip không chỉ pass ngày đầu mà hoạt động đúng sau 5-10 năm trong field.

---

## PHẦN XVII: FILE VÀ FORMAT — HỆ THỐNG TRAO ĐỔI THÔNG TIN GIỮA TOOL

### 17.1 Mỗi File Format Là Ngôn Ngữ Chung Giữa Hai Bước Trong Flow

LEF (Library Exchange Format): physical abstraction của cell và technology. Input cho placement và routing. Định nghĩa pin location, DRC rule, routing layer properties. Cell LEF và Technology LEF là hai phần: technology LEF đến từ foundry, Cell LEF đến từ cell library provider.

DEF (Design Exchange Format): snapshot của physical design tại một thời điểm. Chứa placement của mọi cell instance, routing của mọi net, và power structure. DEF là "bức ảnh" layout có thể đọc được bởi tool khác. Extraction tool cần DEF (hoặc GDS) để know geometry.

SPEF (Standard Parasitic Exchange Format): R và C của mọi net sau extraction. Input cho STA và power analysis. SPEF là cầu nối giữa physical world (wire geometry) và timing world (delay calculation).

SDF (Standard Delay Format): annotated delay từ STA tool sau signoff. Input cho gate-level simulation. SDF là kết quả của STA signoff được "đóng gói" để dùng trong simulation.

GDS (Graphic Data System): full geometric description của layout — tất cả layer, tất cả shape, tất cả cell. GDS là file cuối cùng gửi cho foundry để fabricate (tape-out).

LIB (.lib): timing và power model của standard cell. Chứa delay (NLDM table hoặc CCS), slew table, power table cho mọi input-output arc, và corner-specific values. STA tool cần LIB để biết cell delay bao nhiêu tại given slew và load.

SDC (Synopsys Design Constraints): timing constraint specification. Định nghĩa clock, false path, multicycle path, input/output delay. SDC là "yêu cầu" mà STA compare với "thực tế" từ SPEF + LIB.

ITF (Interconnect Technology File): RC model của metal layer cho extraction tool. Foundry cung cấp per corner. Extraction tool dùng ITF + layout geometry → SPEF.

Mỗi file là output của một step và input cho bước tiếp theo. Sai ở bất kỳ file nào đều lan truyền downstream theo cách của domain đó.

### 17.2 Công Cụ EDA Và Sự Phân Chia Trách Nhiệm

PnR tool (Cadence Innovus, Synopsys ICC2) chịu trách nhiệm từ floorplan đến chip finishing. In-tool timing và DRC là "fast check" trong flow, không phải signoff quality.

Signoff STA tool (Synopsys PrimeTime, Cadence Tempus) chịu trách nhiệm timing signoff với signoff accuracy. Dùng SPEF từ dedicated extraction tool.

Signoff RC Extraction tool (Synopsys StarRC, Cadence Quantus) chịu trách nhiệm extraction chính xác nhất. Dùng ITF từ foundry và layout geometry.

Physical verification tool (Synopsys IC Validator, Mentor Calibre) chịu trách nhiệm DRC, LVS, Antenna với rule deck chính thức từ foundry.

Formal verification tool (Cadence Conformal, Synopsys Formality) chịu trách nhiệm LEC.

Power integrity tool (Cadence Voltus, Synopsys RedHawk) chịu trách nhiệm EM và IR analysis.

Sự phân chia này không phải do historical accident mà vì mỗi domain cần expertise khác nhau và accuracy requirement khác nhau. PnR tool optimize — nó cần speed. Signoff tool verify — nó cần accuracy. Trade-off giữa speed và accuracy là lý do hai loại tool tồn tại song song.

---

## PHẦN XVIII: TỔNG KẾT — MA TRẬN QUAN HỆ TOÀN DIỆN

Phần cuối này tóm tắt quan hệ giữa các khái niệm dưới dạng mô tả văn bản có thứ tự logic, từ foundation đến signoff.

Fabrication process quyết định metal stack properties và là nguồn gốc của mọi physical constraint trong LEF và ITF. LEF mã hóa geometric constraint cho routing tool, ITF mã hóa RC model cho extraction tool. Cell LIB chứa timing model của standard cell, là tham chiếu để STA tính cell delay. SDC chứa timing requirement, là mục tiêu mà design phải đạt. RTL và synthesis netlist là logical starting point trước physical design.

Floorplan quyết định macro location, IO placement, và die area — tạo khung cho routing. Placement quyết định cell location trong core area — tạo điểm đầu và cuối cho mọi net cần route. CTS distribute clock và tạo clock tree với buffer được insert — tạo clock RC và skew profile ảnh hưởng setup và hold margin. Cả ba bước này là prerequisite cho routing.

Global routing phân tích routing resource qua GCELL abstraction. Capacity của GCELL edge là số track có thể dùng. Demand là số net muốn đi qua. Overflow là excess demand vượt capacity. Congestion cao → routing detour → wire length tăng → RC tăng → delay tăng → timing violation. Routing blockage kiểm soát routing resource và bảo vệ sensitive region. Routing constraint và NDR hướng dẫn router về SI, EM, và reliability requirement cho từng net.

Detailed routing thực thi routing guide, tạo actual wire và via. DRC violation sau routing cần được categorize: short và open là showstopper, spacing violation là fixable. Pin access problem là giao điểm giữa placement quality và routing feasibility.

RC extraction dùng actual wire geometry và ITF để tạo SPEF. SPEF là "cầu nối" từ physical world sang timing world. Estimated RC (pre-route) luôn khác actual RC (post-route) — thường actual RC lớn hơn. Sự chênh lệch này là lý do cần post-route timing closure.

Post-route timing closure dùng SPEF thực để fix setup, hold, và DRV violation. Selective re-buffering, cell resizing, critical net re-routing, và useful skew adjustment là các kỹ thuật. ECO flow là vòng lặp STA↔PnR với nguyên tắc minimum-disruption. Crosstalk thêm tầng complexity: SI-aware STA dùng coupling capacitance Cm trong SPEF để tính coupling-adjusted delay.

Power và area recovery khai thác positive slack để downsize cell, swap sang HVT, và remove redundant buffer. Recovery phải không vi phạm timing. VT swap có power saving lớn nhất vì leakage giảm theo hàm mũ với Vt.

Chip finishing gồm filler cell (N-well continuity, power rail continuity), decap cell (dynamic IR drop mitigation), redundant via (reliability, EM margin), wire spreading và widening (lithography yield, EM), và metal fill (CMP uniformity, RC accuracy). Mỗi bước giải quyết một loại failure mode cụ thể.

Signoff cover physical verification (DRC: geometric rule compliance; LVS: connectivity match; Antenna: gate oxide damage prevention; ERC: electrical safety bao gồm ESD và latchup prevention), logical verification (LEC: logic equivalence throughout flow), parasitic extraction (SPEF cho signoff accuracy), STA signoff (timing clean ở tất cả PVT corner với OCV và CPPR; SDF generation cho simulation), EM/IR signoff (long-term reliability và voltage integrity), và gate simulation (dynamic timing anomaly detection). Tất cả phải clean trước tape-out.

SDF từ STA signoff là kết quả cuối cùng của physical implementation được đóng gói để gate simulation sử dụng, khép lại vòng verification từ implementation trở về dynamic behavior check. GDS là layout cuối cùng được gửi cho foundry — kết quả của toàn bộ chuỗi từ RTL đến physical implementation, từ estimated đến actual, từ planning đến verification.

---

Xác nhận nguồn ngoài tài liệu lecture: Các phần mô tả về crosstalk mechanism (coupling capacitance, Miller effect, aggressor-victim model) được xác nhận bởi teamvlsi.com, learnvlsi.com, và chipxpert.in. EM mechanism (void và hillock formation, MTTF) được xác nhận bởi Cadence và Synopsys official documentation. ECO flow và signoff-driven ECO guidance được xác nhận bởi Synopsys whitepaper và EE Times. Global routing GCELL concept và overflow formula được xác nhận bởi signoffsemiconductors.com và ACM ISPD publications. STA concepts (WNS, TNS, OCV, CPPR, SDF, SPEF) được xác nhận bởi Synopsys PrimeTime documentation và Udemy STA course curriculum. File format definitions (LEF, DEF, SPEF, SDF, ITF) được xác nhận bởi ivlsi.com, vlsitalks.com, và Wikipedia LEF/DEF article.
