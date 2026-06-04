Chương 2 — Geometry Foundation: LEF, Placement Grid và Routing Grid
2.1. Mục tiêu của chương

Sau chương này, người học cần hiểu:

Physical Design vận hành trên geometry, không chỉ trên logic graph.
[[LEF]] là nguồn mô tả quan trọng cho placement grid và routing grid.
[[PlacementGrid]] và [[RoutingGrid]] là hai grid khác nhau.
[[Site]], [[Row]], [[Pitch]], [[Track]] là các khái niệm nền tảng, không phải chi tiết phụ.
Placement hợp lệ chưa chắc routing dễ.
Routing hợp lệ phụ thuộc mạnh vào pin location, obstruction, track alignment và metal stack.

Một câu cần nhớ:

Placement đặt cells vào legal sites; Routing nối pins qua legal tracks. Hai hệ grid này phải tương thích thì design mới routable.

2.2. Vì sao phải học geometry trước Placement?

Người mới thường muốn học ngay [[Placement]] hoặc [[Routing]]. Cách học đó dễ bị hổng nền.

Lý do: [[Placement]] không đặt cell ở vị trí tùy ý. Nó phải đặt cell vào các vị trí hợp lệ trên [[PlacementGrid]].
Tương tự, [[Routing]] không vẽ dây tùy ý. Nó phải dùng routing resources trên [[RoutingGrid]] và tuân theo routing rules.

Nếu không hiểu grid, người học sẽ không hiểu các hiện tượng sau:

tại sao cell phải snap vào row;
tại sao cell width thường là bội số của site width;
tại sao pin access có thể fail dù cells đã được đặt hợp lệ;
tại sao macro placement tạo congestion;
tại sao routing blockage ảnh hưởng timing;
tại sao metal pitch ảnh hưởng wire density;
tại sao standard cell architecture phụ thuộc M1 track;
tại sao DEF phải lưu row/track information.

Vì vậy, trước khi học [[Floorplanning]], [[Placement]] và [[Routing]], cần học geometry foundation.

2.3. Hai không gian chính: Placement space và Routing space

Trong Physical Design, có thể tách hai không gian vật lý chính:

Placement space:
- Dành cho instances/cells/macros.
- Được mô tả bằng Site, Row, PlacementGrid.
- Được dùng bởi Floorplanning và Placement.

Routing space:
- Dành cho wires/vias.
- Được mô tả bằng Pitch, Track, RoutingGrid, MetalStack.
- Được dùng bởi Routing và ParasiticExtraction.

Hai không gian này khác nhau nhưng không độc lập.

Nếu placement space không align tốt với routing space, các pins của cell có thể rơi vào vị trí khó access. Khi đó router cần detour, jog hoặc dùng thêm via, dẫn đến:

congestion tăng;
wire length tăng;
capacitance tăng;
[[NetDelay]] tăng;
[[Slew]] xấu hơn;
setup/hold margin xấu hơn;
DRC risk cao hơn.
2.4. [[LEF]] là nguồn sinh ra geometry abstraction

[[LEF]] là nơi nhiều thông tin geometry được định nghĩa cho PnR tool.

Có hai nhóm thông tin quan trọng:

Technology-level geometry:
- routing layers
- metal direction
- pitch
- width/spacing
- via rules
- manufacturing grid
- site definitions

Cell/macro-level geometry:
- cell size
- macro size
- pin shapes
- pin locations
- obstruction regions
- symmetry
- site compatibility

Từ đây, PnR tool xây dựng được hai hệ grid:

Tech LEF LAYER information
  ↓
[[Pitch]]
  ↓
[[Track]]
  ↓
[[RoutingGrid]]

Tech LEF SITE information
  ↓
[[Site]]
  ↓
[[Row]]
  ↓
[[PlacementGrid]]

Đây là quan hệ nền tảng của [[Chain_LEF_to_PnR]].

2.5. [[Pitch]] — khoảng cách lặp của routing resources

[[Pitch]] là khoảng cách đều giữa các routing tracks trên một metal layer.

Trực giác:

Track 0     Track 1     Track 2     Track 3
  |-----------|-----------|-----------|
       Pitch      Pitch      Pitch

Nếu một layer có pitch nhỏ hơn, layer đó có thể có nhiều routing tracks hơn trên cùng một chiều rộng. Nhưng pitch không thể tùy ý giảm vì còn phụ thuộc:

metal width;
spacing rule;
lithography;
resistance;
capacitance;
manufacturing rule;
pin accessibility;
via enclosure;
design rule constraints.
Vì sao Pitch quan trọng?

Pitch ảnh hưởng đến:

số lượng available routing tracks;
routing congestion;
pin access;
metal density;
wire RC;
cell architecture;
standard-cell height;
site alignment.

Ở mức cơ bản, có thể hiểu:

[[Pitch]] quyết định độ mịn của routing grid trên từng metal layer.

Liên kết cần nhớ
[[Pitch]] tạo ra [[Track]].
[[Track]] tạo thành [[RoutingGrid]].
[[RoutingGrid]] được [[Routing]] sử dụng.
[[Pitch]] cũng liên quan gián tiếp đến [[Site]] vì standard-cell architecture thường được thiết kế để align với routing resources.
2.6. [[Track]] — đường routing hợp lệ trên một layer

[[Track]] là một đường routing candidate trên một metal layer.

Nếu [[Pitch]] là khoảng cách, thì [[Track]] là từng vị trí cụ thể.

Có thể mô tả đơn giản:

Track_i = Origin + i × Pitch

Trong đó:

Origin là vị trí bắt đầu;
Pitch là khoảng cách giữa hai tracks;
i là index của track.

Một metal layer có thể có preferred routing direction:

horizontal;
vertical.

Ví dụ phổ biến:

M1: thường dùng local/inter-cell routing, tùy library architecture
M2: horizontal hoặc vertical tùy technology convention
M3: hướng ngược với M2
M4/M5/...: alternating preferred directions

Chi tiết cụ thể phụ thuộc PDK/foundry/tool setup, không nên xem ví dụ trên là quy luật tuyệt đối.

Vì sao Track quan trọng?

Router cần gán wires lên tracks. Nếu nhiều nets cạnh tranh cùng một vùng tracks, sẽ có congestion.

Các yếu tố làm tăng routing demand:

pin density cao;
macro pins dày;
nhiều nets giao nhau;
placement quá chặt;
blockage nhiều;
PDN chiếm nhiều routing resources;
clock routing/shielding chiếm track;
bus hoặc interface signals tập trung một vùng.
Liên kết cần nhớ
[[Pitch]] → [[Track]]
[[Track]] → [[RoutingGrid]]
[[RoutingGrid]] → [[Routing]]
[[Track]] liên quan đến [[Pin]] access và [[Obstruction]]
2.7. [[RoutingGrid]] — không gian routing của design

[[RoutingGrid]] là tập hợp các tracks trên các routing layers.

Có thể hiểu:

RoutingGrid = all routing tracks across all routing layers

[[Routing]] tool dùng [[RoutingGrid]] để tìm đường nối các pins.

Routing không chỉ là “vẽ dây ngắn nhất”. Router phải đồng thời xét:

connectivity;
metal layer direction;
track availability;
via cost;
DRC;
spacing;
congestion;
timing criticality;
crosstalk/noise risk;
power grid obstruction;
macro obstruction;
routing blockage;
antenna rule, nếu flow check;
non-default rules, nếu dùng cho clock hoặc critical nets.
RoutingGrid và timing

RoutingGrid ảnh hưởng timing thông qua wire geometry.

Khi route dài hơn hoặc phải detour:

wire length ↑
  ↓
R_wire, C_wire ↑
  ↓
[[NetDelay]] ↑
  ↓
[[StageDelay]] ↑
  ↓
[[Slack]] có thể xấu hơn

Vì vậy, routing grid không chỉ là vấn đề physical legality. Nó ảnh hưởng trực tiếp đến [[STA]].

2.8. [[Site]] — đơn vị placement hợp lệ

[[Site]] là đơn vị lưới cơ bản để đặt standard cell.

Một standard cell thường có:

height tương thích với site height;
width là bội số của site width;
pins được thiết kế để router có thể access;
power rail alignment phù hợp với row orientation.

Có thể hiểu đơn giản:

Site = atomic legal placement tile

Các cells không được đặt tùy ý theo tọa độ liên tục. Chúng phải được snap vào site positions hợp lệ.

Vì sao Site quan trọng?

Nếu không có [[Site]], placement tool không biết:

cell có được đặt tại một vị trí không;
cell có align với power rail không;
cell có align với row không;
cell có vi phạm manufacturing/legal placement rule không;
multi-height cell có tương thích row structure không.
Site và standard-cell architecture

[[Site]] không chỉ là một ô hình học. Nó phản ánh architecture của standard-cell library.

Ví dụ:

single-height cells cần row height phù hợp;
multi-height cells cần nhiều row hoặc special row structure;
power rail của cells phải align với row;
cell symmetry ảnh hưởng legal orientation.
Liên kết cần nhớ
[[LEF]] định nghĩa [[Site]].
[[Site]] tile theo chiều ngang để tạo [[Row]].
[[Row]] tạo thành [[PlacementGrid]].
[[Placement]] đặt [[StandardCell]] lên [[PlacementGrid]].
2.9. [[Row]] — dải placement hợp lệ cho standard cells

[[Row]] là một dải gồm nhiều [[Site]] lặp lại theo chiều ngang trong core area.

Có thể hiểu:

Row = repeated Sites across a horizontal legal placement band

Rows thường được tạo trong [[Floorplanning]], dựa trên:

core boundary;
site definition;
row height;
row orientation;
macro blockage;
placement blockage;
power rail convention.
Vai trò của Row

[[Placement]] dùng rows để biết standard cells được phép đặt ở đâu.

Một row có thể bị cắt hoặc phân mảnh bởi:

macro;
hard blockage;
reserved channel;
power structure;
keepout;
boundary constraint.

Row fragmentation làm giảm effective placeable area và có thể làm placement khó hơn.

Row orientation

Rows thường có orientation pattern để power rails và well structures align đúng.
Ví dụ một số flow có thể dùng alternating orientation như R0/MY, hoặc convention khác tùy tool/library.

Điểm quan trọng:

Row orientation không phải chi tiết hình thức. Nó ảnh hưởng legal placement, rail alignment và cell abutment.

Không nên ghi nhớ cứng một naming convention duy nhất. Tool khác nhau có thể dùng cách gọi khác nhau.

Liên kết cần nhớ
[[Site]] → [[Row]]
[[Row]] → [[PlacementGrid]]
[[Row]] được tạo trong [[Floorplanning]]
[[Row]] được dùng bởi [[Placement]] và [[DetailedPlacement]]
2.10. [[PlacementGrid]] — không gian đặt standard cells

[[PlacementGrid]] là tập hợp các legal site positions trong rows.

Có thể hiểu:

PlacementGrid = all legal Site positions inside all legal Rows

[[Placement]] hoạt động trên [[PlacementGrid]], không hoạt động trên không gian liên tục.

Một placement hợp lệ cần thỏa:

cell nằm trên legal row;
cell align với legal site;
cell không overlap cell khác;
cell không nằm trong hard blockage;
orientation hợp lệ;
macro/fixed object constraint được tôn trọng;
power rail alignment hợp lệ;
multi-height rule hợp lệ nếu có.
PlacementGrid và utilization

Khái niệm utilization phải được hiểu trên placeable area, không chỉ core area.

Một công thức trực giác:

Standard Cell Utilization =
Total Standard Cell Area / Placeable Standard Cell Area

Trong đó placeable area bị giảm bởi:

macro area;
placement blockage;
halo/keepout;
unusable slivers;
reserved channels;
row fragmentation;
power/grid constraints;
boundary effects.

Vì vậy, một core nhìn có vẻ rộng nhưng effective placeable area có thể nhỏ hơn nhiều.

Người mới thường hiểu sai

Sai lầm phổ biến:

“Core utilization = cell area / core area là đủ.”

Cách hiểu đúng:

Cần phân biệt raw core area và actual placeable area. Macro, blockage, halo và row fragmentation có thể làm effective utilization cao hơn nhiều so với con số ban đầu.

2.11. [[CellAbstract]] — nơi placement và routing gặp nhau

[[CellAbstract]] là abstract physical view của standard cell hoặc macro trong [[LEF]].

Nó thường chứa:

size;
origin;
symmetry;
site compatibility;
[[Pin]] shapes/locations;
[[Obstruction]] regions.

[[CellAbstract]] là điểm giao giữa placement và routing:

Placement cần:
- cell size
- legal site compatibility
- boundary

Routing cần:
- pin location
- pin shape
- obstruction

Nếu cell abstract không chính xác, PnR tool có thể:

đặt cell sai;
route vào vùng không được phép;
không access được pin;
tạo DRC violation;
tạo mismatch với GDS signoff.
Standard cell và hard macro

Với [[StandardCell]], [[CellAbstract]] thường nhỏ, nhiều instance, được placement tự động trên rows.

Với [[HardIP]] hoặc macro:

kích thước lớn;
có nhiều [[Pin]];
có nhiều [[Obstruction]];
thường được đặt ở [[Floorplanning]];
vị trí thường fixed trước placement;
ảnh hưởng mạnh đến congestion, timing, PDN và routing channel.
2.12. [[Pin]] — điểm kết nối mà router phải access

[[Pin]] trong LEF/cell abstract là hình học cho biết net có thể connect vào cell/macro ở đâu.

Pin không chỉ là tên logic. Trong Physical Design, pin có geometry:

layer;
shape;
location;
access direction;
relation to tracks;
possible access points.

Router phải tìm đường từ pin này đến pin khác qua routing grid.

Pin access

Pin access có thể khó nếu:

pins quá dày;
pins nằm lệch track;
macro pins tập trung một cạnh;
obstruction che vùng routing;
standard cells đặt quá sát nhau;
local routing resources bị chiếm;
PDN hoặc blockage làm mất tracks;
placement tạo pin-density hotspot.

Pin access issue là một trong những cầu nối quan trọng giữa placement và routing.

Người mới thường hiểu sai

Sai lầm phổ biến:

“Nếu cells nối với nhau trong netlist thì router sẽ nối được.”

Cách hiểu đúng:

Router chỉ nối được nếu pins có thể access được qua legal routing resources. Connectivity logic không đảm bảo routability vật lý.

2.13. [[Obstruction]] — vùng router không được dùng

[[Obstruction]] là vùng trong cell/macro abstract mà external router không được route xuyên qua.

Obstruction có thể đại diện cho:

internal metal của macro;
pre-routed wires;
protected regions;
reserved layers;
vùng không cho signal routing.

Với standard cell, obstruction thường nhỏ.
Với macro hoặc [[HardIP]], obstruction có thể rất lớn và ảnh hưởng mạnh đến global routing.

Obstruction và macro placement

Nếu macro đặt không tốt, obstruction của macro có thể:

cắt routing channel;
tạo congestion quanh macro corner;
làm pin access khó;
buộc nets detour;
tăng wire length;
tăng [[NetDelay]];
làm timing xấu;
làm routing DRC khó clean.

Vì vậy, [[MacroPlacement]] không chỉ là đặt macro gần các logic liên quan. Nó còn phải tạo đủ routing channel và tránh pin-density/congestion hotspot.

2.14. Alignment giữa PlacementGrid và RoutingGrid

Đây là ý chính của chương 2.

Placement grid và routing grid phải tương thích:

[[Site]] / [[Row]] / [[PlacementGrid]]
      ↕ must align
[[Pitch]] / [[Track]] / [[RoutingGrid]]

Lý do:

standard cell pins phải nằm ở vị trí router có thể access;
cell boundary phải tương thích với M1/M2 routing resources;
row structure phải hỗ trợ rail alignment;
routing tracks phải đi qua hoặc gần pin access points;
legal placement phải không phá pin accessibility.

Nếu không align, design vẫn có thể có placement hợp lệ nhưng routing xấu.

Hậu quả thường gặp:

pin access fail;
detour nhiều;
via count tăng;
congestion hotspot;
timing degradation;
DRC violation;
routing runtime tăng;
post-route QoR kém.
Câu cần nhớ

Legal placement không đồng nghĩa với routable placement.

Một placement có thể legal theo site/row nhưng vẫn không routable vì pin access và routing resources không đủ.

2.15. Từ geometry sang các bước sau

Chương 2 là nền cho các chương sau:

[[LEF]]
  ↓
[[Pitch]] → [[Track]] → [[RoutingGrid]] → [[Routing]]
  ↓
[[Site]] → [[Row]] → [[PlacementGrid]] → [[Placement]]
  ↓
[[CellAbstract]] → [[Pin]] + [[Obstruction]]
  ↓
[[Floorplanning]] + [[Placement]] + [[Routing]]

Cụ thể:

[[Floorplanning]] dùng geometry để tạo core, rows, tracks, macro placement, blockages và PDN planning.
[[Placement]] dùng [[PlacementGrid]] để đặt standard cells hợp lệ.
[[Routing]] dùng [[RoutingGrid]] để nối pins.
[[ParasiticExtraction]] dùng final wire geometry để tạo [[SPEF]].
[[STA]] dùng [[SPEF]] để tính [[NetDelay]] chính xác hơn.
[[Signoff]] kiểm tra design cuối cùng có sạch không.
2.16. Những hiểu lầm cần loại bỏ sau chương 2
Hiểu lầm 1

“LEF chỉ là file phụ.”

Sửa lại:

[[LEF]] là một trong các input vật lý nền tảng nhất. Nó định nghĩa abstract cell geometry, pin, obstruction, placement site và routing technology information.

Hiểu lầm 2

“Placement grid và routing grid là một.”

Sửa lại:

[[PlacementGrid]] dành cho cell placement. [[RoutingGrid]] dành cho wire routing. Chúng khác nhau nhưng phải align.

Hiểu lầm 3

“Cell đặt hợp lệ thì routing sẽ ổn.”

Sửa lại:

Cell placement hợp lệ chỉ là điều kiện cần. Routability còn phụ thuộc pin access, routing resources, congestion, blockages, macro placement và PDN usage.

Hiểu lầm 4

“Pin là khái niệm logic.”

Sửa lại:

Trong Physical Design, [[Pin]] là geometry mà router phải access được. Pin location và pin shape ảnh hưởng trực tiếp đến routing và timing.

Hiểu lầm 5

“Macro chỉ chiếm area.”

Sửa lại:

Macro còn tạo obstruction, pin-density hotspot, routing detour, PDN challenge và timing/clock-distribution constraint.

2.17. Tổng kết chương 2

Các quan hệ phải thuộc:

[[LEF]] → [[Pitch]] → [[Track]] → [[RoutingGrid]] → [[Routing]]

[[LEF]] → [[Site]] → [[Row]] → [[PlacementGrid]] → [[Placement]]

[[GDS]] / full layout → abstract generation → [[CellAbstract]]

[[CellAbstract]] → [[Pin]] + [[Obstruction]]

[[PlacementGrid]] ↔ [[RoutingGrid]] alignment → routability

Chương 1 giải thích tool cần input gì.
Chương 2 giải thích tool hiểu không gian vật lý như thế nào.

Sau hai chương này, người học đã có nền để học tiếp:

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