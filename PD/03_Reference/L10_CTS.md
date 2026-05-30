# Clock Tree Synthesis (CTS) — Part II: Phân tích Lý thuyết Toàn diện

> **Nguồn**: Lecture 10 — Tresemi (Phil Hoang), tích hợp xác minh khái niệm từ Lecture 9.
> Phạm vi Part II: CTS Optimization Techniques · CTS Flow · Post-CTS Optimization · CTS Outputs & Quality Review · Thiết lập CTS trên Design mới · Clock Tree Debugger · Inverter-Based vs Buffer-Based Clock Trees · Virtual Clock · Generated Clock.

---

## Mục lục

1. [Vị trí của Part II trong toàn bộ CTS Flow](#1)
2. [CTS Optimization Techniques](#2)
3. [CTS Execution Flow — 4 giai đoạn nội tại](#3)
4. [Post-CTS Optimization](#4)
5. [CTS Outputs và Quality Review](#5)
6. [Thiết lập CTS trên Design mới — Three-Stage Flow](#6)
7. [Clock Tree Debugger (CTD)](#7)
8. [Inverter-Based vs Buffer-Based Clock Trees](#8)
9. [Virtual Clock và Generated Clock](#9)
10. [Mối liên kết tổng hợp toàn bộ L9 + L10](#10)

---

<a name="1"></a>
## 1. Vị trí của Part II trong toàn bộ CTS Flow

Part I (L9) đã thiết lập nền tảng lý thuyết: mục tiêu CTS, input, pre-CTS sanity check, các thuật ngữ cốt lõi (insertion delay, skew, slew, duty cycle, OCV, CPPR…), exception pin, CTS Spec, và các kiến trúc clock tree. Part II (L10) tiếp tục từ bước **CTS Optimization Techniques** — kỹ thuật tối ưu hóa mà tool áp dụng trong quá trình thực thi — rồi đi sâu vào bốn giai đoạn nội tại của lệnh `ccopt_design`, sau đó là post-CTS optimization, quality review, và các công cụ hỗ trợ debug.

---

<a name="2"></a>
## 2. CTS Optimization Techniques

Khi CTS tool xây dựng và cân bằng clock tree, nó không chỉ đơn thuần insert buffer. Tool có thể áp dụng nhiều kỹ thuật tối ưu hóa khác nhau. Một số kỹ thuật (Buffering, Cell Sizing, Cloning, VT-Swapping, Logic Restructuring) đã được phân tích trong Placement lecture vì bản chất của chúng áp dụng rộng hơn. Các kỹ thuật đặc thù cho CTS gồm:

---

### 2.1 Cell Relocation

**Định nghĩa và cơ chế:** Cell Relocation là kỹ thuật điều chỉnh vị trí vật lý của clock cell (thường là clock buffer hoặc inverter) để cân bằng lại phân phối tải tại output của cell đó.

**Nguyên lý:**
Trong một clock tree, một clock buffer đôi khi có output load không đều — một số sink nằm xa về một phía, số khác nằm gần phía còn lại. Điều này gây ra sự chênh lệch wire delay từ buffer đến các nhóm sink khác nhau, dẫn đến skew tăng. Bằng cách dịch chuyển buffer về vị trí gần trung tâm của cluster sink mà nó drive, tool giảm sự bất đồng đều này.

**Mối liên hệ với L9:** Phần 5.2 (Clock Skew) trong L9 chỉ ra rằng một trong các nguyên nhân gây skew là *chênh lệch độ dài clock path* và *clock loading imbalance*. Cell Relocation trực tiếp giải quyết cả hai nguyên nhân này đồng thời.

**Hệ quả:** Sau khi re-locate, insertion delay đến từng nhóm sink trở nên đều hơn, skew giảm mà không cần insert thêm buffer hay tăng độ sâu cây.

---

### 2.2 Level Adjustment

**Định nghĩa và cơ chế:** Level Adjustment là kỹ thuật di chuyển clock sink (Flip-flop) đến các mức khác nhau trong phân cấp clock tree, nhằm cân bằng clock arrival time và giảm skew.

**Nguyên lý:**
Clock tree có cấu trúc phân cấp (hierarchical). Một Flip-flop ở *level thấp* (gần root) sẽ nhận clock sớm hơn Flip-flop ở *level cao* (gần leaf, xa root hơn). Nếu một số Flip-flop có insertion delay bất thường — quá ngắn hoặc quá dài so với nhóm — tool có thể kéo chúng lên hoặc đẩy xuống một level trong cây để điều chỉnh arrival time mà không cần thay đổi cấu trúc tổng thể của tree.

**Phân biệt với Cell Relocation:** Cell Relocation thay đổi *vị trí tọa độ vật lý* (x, y) của cell trong layout. Level Adjustment thay đổi *mức tầng logic* (tree depth) của cell trong clock tree hierarchy — tức là quyết định cell sẽ được drive bởi buffer nào (ở level nào).

**Điều kiện áp dụng:** Thường áp dụng khi một cluster có Flip-flop phân bố không đều về mặt insertion delay, đặc biệt khi có Flip-flop ở các vùng xa trung tâm clock domain.

---

### 2.3 Reconfiguration

**Định nghĩa và cơ chế:** Reconfiguration là kỹ thuật điều chỉnh lại *clustering* và *connectivity* (kết nối topology) của các Flip-flop trong clock tree, nhằm cải thiện load balancing, giảm skew, và tối ưu hóa clock distribution.

**Nguyên lý:**
Trong giai đoạn đầu của CTS, tool tự động nhóm các Flip-flop vào cluster dựa trên vị trí vật lý và các tiêu chí DRV. Sau khi routing và post-conditioning, kết quả có thể cho thấy một số cluster có load không đều — một buffer drive quá nhiều Flip-flop trong khi buffer kề bên drive ít. Reconfiguration cho phép tool tái tổ chức lại phân vùng này: chuyển một số Flip-flop từ cluster này sang cluster kia để cân bằng tải.

**Điểm khác biệt so với Level Adjustment:** Reconfiguration thay đổi *sự phân công sink cho từng sub-tree*, trong khi Level Adjustment thay đổi *độ sâu tầng* của từng sink trong cây hiện tại.

**Khi nào áp dụng:** Khi `report_ccopt_skew_groups` cho thấy một số skew group có insertion delay phân bổ lệch, hoặc khi clock DAG (Directed Acyclic Graph) có nhánh không cân bằng về số lượng sink.

---

### 2.4 Dummy Load Insertion

**Định nghĩa và cơ chế:** Dummy Load Insertion là kỹ thuật bổ sung *artificial load* vào các nhánh clock tree ngắn để cân bằng clock loading, khớp insertion delay giữa các nhánh, và đều hóa clock path delay.

**Nguyên lý:**
Trong một clock tree, nếu hai nhánh từ cùng một buffer driver có chiều dài rất khác nhau, thì clock arrival time tại sink cuối của hai nhánh sẽ chênh lệch đáng kể. Thay vì cố gắng rút ngắn nhánh dài (không thể vì số lượng sink là cố định), tool *thêm tải giả* vào nhánh ngắn hơn để kéo dài insertion delay của nhánh đó, làm cho hai nhánh có delay tương đương.

Dummy load được thêm vào *clock path ngắn nhất* để khớp với clock path dài hơn:

$$\Delta T_{\text{dummy}} \approx T_{\text{longest\_path}} - T_{\text{shortest\_path}}$$

**Ba loại Dummy Load:**

| Loại | Cơ chế vật lý | Đặc điểm |
|---|---|---|
| **Buffer-Based** | Sử dụng thêm clock buffer | Tạo delay thêm và tăng capacitance; có gate delay + wire delay |
| **Inverter-Based** | Sử dụng inverter cell | Kiểm soát delay và load chính xác hơn; thường dùng khi cần thêm inversion stage |
| **RC-Based** | Sử dụng wire RC | Delay thuần từ resistance và capacitance của wire; không cần cell |

**Hệ quả về power:** Dummy load là tải không drive bất kỳ logic nào — nó chỉ tồn tại để cân bằng timing. Do đó nó tiêu thụ switching power mà không đóng góp gì cho chức năng. CTS tool phải cân nhắc trade-off giữa skew cải thiện và power tăng.

**Liên hệ với L9:** L9 đề cập đến mục tiêu "tối thiểu hóa power consumption" — Dummy Load Insertion đi ngược lại mục tiêu này nếu dùng quá nhiều. Đây là lý do tại sao nó chỉ được dùng như phương án cuối cùng khi các kỹ thuật khác không đủ hiệu quả.

---

### 2.5 Load Splitting

**Định nghĩa và cơ chế:** Load Splitting là kỹ thuật chia các *large fanout load* thành các nhóm nhỏ hơn bằng cách insert thêm buffer hoặc nhân bản (clone) driver, nhằm cải thiện clock transition quality và thỏa mãn DRV constraints.

**Nguyên lý:**
Một clock buffer drive quá nhiều sink (fanout lớn) sẽ gặp hai vấn đề:
1. **Max fanout violation**: vượt quá giới hạn fanout cho phép của cell library.
2. **Slew degradation**: output slew của driver tăng tỷ lệ với tổng output capacitance. Với fanout quá lớn, slew quá chậm → vi phạm max transition constraint → gây DRV.

Load Splitting giải quyết bằng hai phương án:

**Load splitting via Cloning:** Nhân bản driver hiện tại thành hai (hoặc nhiều) bản sao. Mỗi bản sao drive một subset của tổng fanout ban đầu. Cả hai driver drive cùng một input net (kết nối song song ở đầu vào). Đây là kỹ thuật mở rộng trực tiếp từ *Cloning* đã học ở Placement lecture, áp dụng cụ thể cho clock tree.

**Load splitting via Buffering:** Thay vì nhân bản driver, insert thêm một tầng buffer trung gian. Driver gốc bây giờ chỉ drive các buffer mới, mỗi buffer mới drive một phần subset nhỏ hơn của tổng fanout.

**Điểm khác biệt giữa hai phương án:**
- Cloning không tăng tree depth nhưng chiếm thêm placement area.
- Buffering tăng tree depth một level nhưng có thể tận dụng vị trí hiện có tốt hơn và giảm wire length trung bình.

**Liên hệ với DRV concepts từ L9:** L9 phần 4 (Pre-CTS Sanity Checks) yêu cầu max fanout, max transition, max capacitance phải được clean *trước khi* CTS, ngoại trừ trên clock net. Load Splitting là cơ chế CTS tool dùng để clean DRV *trên clock net* trong quá trình synthesis.

---

### 2.6 Useful Skew

**Định nghĩa tổng quát:** Useful Skew là kỹ thuật *cố ý tạo ra* sự chênh lệch clock arrival time giữa launch Flip-flop và capture Flip-flop trên một timing path cụ thể, nhằm cải thiện setup hoặc hold timing mà không cần thay đổi logic datapath.

**Nguyên lý vật lý:**

Trong L9 (phần 5.4 và 5.5), setup slack và hold slack được định nghĩa như sau:

$$\text{Setup Slack} = T_{\text{cycle}} + T_{\text{skew}} - T_{\text{su}} - T_{\text{data\_path}}$$

$$\text{Hold Slack} = T_{\text{data\_path}} - T_{\text{hold}} - T_{\text{skew}}$$

Trong đó $T_{\text{skew}} = T_{\text{capture\_arrival}} - T_{\text{launch\_arrival}}$.

Khi $T_{\text{skew}} > 0$ (positive skew — capture đến sau launch): setup slack **tăng**, hold slack **giảm**.
Khi $T_{\text{skew}} < 0$ (negative skew — capture đến trước launch): setup slack **giảm**, hold slack **tăng**.

Useful Skew khai thác quan hệ này: với các path đang bị **setup violation**, tool sẽ *cố ý làm capture clock đến muộn hơn* (positive skew) để tăng setup margin. Với các path đang bị **hold violation**, tool sẽ *cố ý làm capture clock đến sớm hơn* (negative skew) để tăng hold margin.

**Ví dụ minh họa định lượng (từ slide L10):**

Xét chuỗi: FF1 → combinational logic 9ns → FF2 → combinational logic 5ns → FF3.  
Tất cả Flip-flop có $T_{cp} = 1\text{ns}$ (clock-to-Q), $T_{su} = 1\text{ns}$, $T_{hold}$ bỏ qua cho đơn giản.  
Clock period $= 10\text{ns}$. Clock arrival tại tất cả FF (trước khi dùng useful skew) $= 2\text{ns}$.

**Path FF1 → FF2** (data path delay = 9ns):

$$\text{Arrival time} = T_{\text{launch}} + T_{cp} + T_{dp} = 2 + 1 + 9 = 12\text{ns}$$

$$\text{Required time} = T_{\text{cycle}} + T_{\text{capture}} - T_{su} = 10 + 2 - 1 = 11\text{ns}$$

$$\text{Setup Slack} = 11 - 12 = -1\text{ns} \quad \Rightarrow \text{VIOLATED}$$

**Path FF2 → FF3** (data path delay = 5ns):

$$\text{Arrival time} = 2 + 1 + 5 = 8\text{ns}$$

$$\text{Required time} = 10 + 2 - 1 = 11\text{ns}$$

$$\text{Setup Slack} = 11 - 8 = +3\text{ns} \quad \Rightarrow \text{MET, có dư 3ns}$$

**Chiến lược Useful Skew:** Path FF1→FF2 cần +1ns slack. Path FF2→FF3 có +3ns dư. Tool có thể làm clock đến FF2 muộn hơn 1–2ns (thay vì 2ns, làm 3–4ns). Điều này:
- Với FF1→FF2: capture đến muộn hơn → $T_{\text{skew}}$ tăng → setup slack tăng từ -1ns lên ≥ 0ns.
- Với FF2→FF3: bây giờ launch (FF2) cũng đến muộn hơn tương đương, setup slack giảm xuống nhưng vẫn dương do có dư 3ns.

**Quan hệ Useful Skew với Hold Timing:** Thay đổi skew để cải thiện setup của path này có thể làm xấu hold của path khác. Tool phải phân tích đồng thời setup và hold khi áp dụng useful skew — đây là lý do nó cần được kiểm soát cẩn thận và thường chỉ được bật tường minh bằng `set_ccopt_property useful_skew true`.

**Phân biệt với skew thông thường:** Trong CTS thông thường, skew là thứ cần *minimize*. Trong Useful Skew, một lượng skew có kiểm soát được *cố ý tạo ra* để phục vụ mục đích timing closure. Useful Skew không phải là bug mà là công cụ.

---

### 2.7 Buffering và Cell Sizing (từ Placement lecture, áp dụng lại trong CTS)

Hai kỹ thuật này đã được đề cập trong Placement lecture nhưng vẫn được áp dụng trong CTS context:

**Buffering:** Insert clock buffer vào giữa driver và load để cải thiện slew, giảm fanout, và tăng drive strength. Trong CTS, buffer được chọn từ danh sách được chỉ định trong CTS Spec (buffer_cells, inverter_cells).

**Cell Sizing:** Thay thế cell hiện tại bằng cell cùng loại nhưng có drive strength khác (ví dụ từ CLKBUF_X2 sang CLKBUF_X4). Sizing up tăng drive strength → giảm slew → cải thiện transition. Sizing down giảm power và area nhưng có thể làm xấu slew.

**VT-Swapping (Vt-Swapping):** Thay thế cell có Threshold Voltage cao (High-VT) bằng cell cùng logic nhưng có Threshold Voltage thấp hơn (Standard-VT hoặc Low-VT) để giảm delay, hoặc ngược lại để giảm leakage power. Có trade-off trực tiếp giữa delay và leakage power:
- High-VT: delay cao, leakage thấp.
- Low-VT: delay thấp, leakage cao.
VT-Swapping trong CTS thường được áp dụng trên critical clock path để giảm insertion delay hoặc trên non-critical path để tiết kiệm power.

**Logic Restructuring:** Tái tổ chức cấu trúc logic của combinational path để giảm critical path delay. Áp dụng cho data path, không trực tiếp trên clock path, nhưng ảnh hưởng gián tiếp đến CTS vì data path timing phải đáp ứng sau khi clock tree được built.

---

<a name="3"></a>
## 3. CTS Execution Flow — 4 Giai đoạn Nội tại

Khi lệnh thực thi CTS (`ccopt_design`) được gọi, tool thực hiện bốn giai đoạn nội tại theo thứ tự:

$$\text{Clustering} \rightarrow \text{Balancing} \rightarrow \text{Routing Clock Tree} \rightarrow \text{Post-Conditioning}$$

Mỗi giai đoạn có mục tiêu riêng, và kết quả của giai đoạn trước là input cho giai đoạn sau. Đây là CTS Flow thực sự diễn ra bên trong tool.

---

### 3.1 Giai đoạn 1: Clustering

**Định nghĩa:** Clustering là giai đoạn CTS tool nhóm các clock sink (Flip-flop, ICG, Macro clock pin) liên quan thành các cluster trước khi xây dựng clock tree thực sự.

**Tiêu chí phân nhóm:** Tool dựa trên:
- **Vị trí vật lý (spatial proximity):** Các sink gần nhau về tọa độ (x, y) trong layout được nhóm cùng cluster.
- **Fanout:** Mỗi cluster phải có số lượng sink phù hợp để buffer driver không vi phạm max fanout.
- **Capacitance:** Tổng capacitance của cluster phải dưới ngưỡng max capacitance.
- **DRV constraints:** Tool insert buffer vào đúng chỗ để đảm bảo tất cả max transition, max capacitance, max fanout được thỏa mãn *ngay từ giai đoạn này*.

**Ý nghĩa của Clustering trong clock tree hierarchy:**
Mỗi cluster sẽ tương ứng với một sub-tree của clock tree. Cluster = nhóm Flip-flop chia sẻ cùng một clock buffer tại lá (leaf buffer). Từ các cluster này, tool xây dựng cấu trúc trunk-leaf (xem L9 phần NDR).

**Ba phương pháp Clustering:**

| Phương pháp | Hướng đi | Đặc điểm |
|---|---|---|
| **Top-Down Clustering** | Từ một cluster lớn chứa tất cả sink → chia đệ quy thành cluster nhỏ hơn | Phù hợp với thiết kế lớn; dễ kiểm soát global structure |
| **Bottom-Up Clustering** | Từ từng sink riêng lẻ → gộp dần thành cluster lớn hơn | Tối ưu local clustering tốt hơn |
| **Iterative Clustering** | Kết hợp top-down và bottom-up lặp đi lặp lại | Chất lượng clock tree cao nhất; tốn thời gian hơn |

**Output của Clustering:** Clock tree được built với các buffer được insert để thỏa mãn DRV. Số lượng buffer, inverter, ICG cell được report ra. Không có skew balancing tại giai đoạn này.

**Đọc Clustering Log:**
```
Clock DAG stats before clustering:  b=0, i=0, icg=34, l=2, total=36
Clock DAG stats after 'Clustering': b=209, i=0, icg=34, l=2, total=245
```
Diễn giải: Trước clustering không có buffer. Sau clustering, 209 buffer được insert (b=209) để thỏa mãn DRV. Số ICG (34) và logic cell (2) không thay đổi vì chúng là pre-placed clock gating cell.

---

### 3.2 Giai đoạn 2: Balancing

**Định nghĩa:** Balancing là giai đoạn điều chỉnh clock path để **cân bằng clock arrival time** giữa các sink, tối thiểu hóa clock skew.

**Cơ chế hoạt động:**

Sau Clustering, clock tree đã được built về mặt cấu trúc nhưng insertion delay đến từng sink có thể rất khác nhau. Balancing sử dụng *skew group constraints* (được định nghĩa trong CTS Spec, xem L9 phần 7) như mục tiêu tối ưu hóa.

Tool lặp đi lặp lại, mỗi iteration thực hiện một trong các tác động:
- Insert thêm buffer để tăng delay trên path ngắn.
- Điều chỉnh clock tree topology (re-clustering cục bộ).
- Điều chỉnh độ sâu nhánh (Level Adjustment).
- Áp dụng Useful Skew nếu được cho phép.

**Điều kiện dừng:**
- Tất cả skew group đạt target skew; **hoặc**
- Không có cải thiện thêm nào khả thi (convergence).

**Thực chất của Balancing:** Balancing là bài toán tối ưu hóa có ràng buộc: tối thiểu hóa $\max(T_{\text{sink}_i}) - \min(T_{\text{sink}_j})$ cho từng skew group, trong khi thỏa mãn DRV constraints và area/power budget.

**Đọc Balancing Log:**
```
Clock DAG stats after clustering:   b=209  (từ Clustering)
Clock DAG stats after Improving fragments clock skew: b=211  (chỉ thêm 2 buffer)
```
Diễn giải: Trong ví dụ này, Balancing chỉ cần insert thêm 2 buffer. Đây là dấu hiệu tốt — design đã khá balanced sau Clustering, hoặc CTS Spec đã được tối ưu tốt.

---

### 3.3 Giai đoạn 3: Routing Clock Tree

**Định nghĩa:** Giai đoạn này thực hiện *detailed routing* cho tất cả clock net, sử dụng các routing rule đặc biệt (NDR, shielding).

**Tại sao clock net cần routing rule riêng:**
Như đã phân tích trong L9 (phần 9 — NDR Rules), clock net nhạy cảm hơn data net về timing và SI. Sau khi clock tree đã được balanced trong không gian lý tưởng (pre-route), routing thực tế sẽ thêm vào wire RC, coupling capacitance, và via resistance. Những yếu tố này phải được kiểm soát bằng NDR.

**Phân cấp routing theo loại net:**

$$\text{Top net} \supset \text{Trunk net} \supset \text{Leaf net}$$

| Loại net | Vị trí trong cây | Metal layer điển hình | NDR điển hình |
|---|---|---|---|
| **Top** | Từ clock source đến nhánh chính đầu tiên | Layer cao nhất (M6–M8) | 2x width, 2x spacing, shielded |
| **Trunk** | Các nhánh trung gian | Layer trung (M5–M6) | 2x width, 2x spacing, có thể shielded |
| **Leaf** | Từ leaf buffer đến Flip-flop | Layer thấp hơn (M4–M5) | Nhẹ hơn, thường 1x với spacing tăng |

**Tại sao dùng metal layer cao hơn cho clock net:**
- Resistance thấp hơn (wire rộng hơn, material như copper có resistivity thấp).
- Coupling capacitance với neighbor nets thấp hơn (pitch rộng hơn ở higher layers).
- Tốc độ propagation nhanh hơn → insertion delay thấp hơn và variation nhỏ hơn.
- Ít bị ảnh hưởng bởi congestion từ Standard Cell (nằm ở lower layers).

**Multi-cut vias:** Clock routing ưu tiên dùng multi-cut via (via có nhiều điểm tiếp xúc thay vì một) vì:
- **Reliability cao hơn**: nếu một cut bị lỗi sản xuất, cut kia vẫn duy trì kết nối.
- **Electromigration (EM) robustness tốt hơn**: dòng điện được chia đều cho nhiều cut → mật độ dòng/cut thấp hơn.
- **Via resistance thấp hơn**: nhiều cut song song → tổng resistance thấp hơn.

**Output của giai đoạn Routing:**
```
Clock DAG stats after routing clock trees:
  wire lengths: trunk=10093um, leaf=56726um, total=66819um
  Remaining Transition: {count=8, worst=0.083ns}
```
Sau routing, có 8 transition violation còn lại — đây là input cho Post-Conditioning.

---

### 3.4 Giai đoạn 4: Post-Conditioning

**Định nghĩa:** Post-Conditioning là giai đoạn tối ưu hóa CTS cuối cùng, nơi *propagated clock delay* (delay thực từ routed net) được sử dụng để phát hiện và sửa chữa các violation còn lại sau routing.

**Sự thay đổi quan trọng so với các giai đoạn trước:**
Trong Clustering và Balancing, tool làm việc với *estimated delay* (tính toán dựa trên wire model). Sau Routing, tool có thể trích xuất *RC parasitics* thực từ routed net, cho ra *extracted delay* chính xác hơn nhiều. Post-Conditioning sử dụng delay chính xác này làm basis cho optimization.

**Điều gì xảy ra sau routing mà cần sửa:**
- Wire routing thực tế thêm vào RC delay, coupling capacitance, via resistance không hoàn toàn khớp với estimate.
- Clock paths bị detour (đi vòng) do congestion → tăng RC → tăng insertion delay và slew.
- Coupling capacitance từ aggressor clock/data net gây transition violation.

**Các loại violation được sửa trong Post-Conditioning:**
- Setup violation (từ data path affected bởi clock timing thay đổi)
- Hold violation
- Transition violation (max slew)
- Capacitance violation (max cap)
- Fanout violation (max fanout)

**CPPR (Clock Path Pessimism Removal):** Đây là điều chỉnh quan trọng sau routing. Trong STA, khi tính setup slack, phần clock path chung (common path — phần clock đi qua cùng net/buffer cho cả launch và capture) được tính hai lần với worst-case corner (một lần cho launch, một lần cho capture). Đây là *pessimism* không có thực. CPPR loại bỏ sự bi quan này bằng cách cộng lại phần delay bị tính trùng:

$$\text{Setup Slack}_{\text{with CPPR}} = \text{Setup Slack}_{\text{without CPPR}} + T_{\text{CPPR}}$$

Giá trị $T_{\text{CPPR}}$ dương → slack thực tế tốt hơn so với tính mà không có CPPR. Post-Conditioning hiệu chỉnh timing analysis với CPPR để có kết quả chính xác hơn trước khi quyết định có cần insert thêm buffer không.

**Phân biệt Insertion Delay vs Clock Latency (quan trọng từ slide 37):**

| Khái niệm | Nguồn gốc | Thời điểm được xác định | Thay đổi sau routing? |
|---|---|---|---|
| **Insertion Delay** | Snapshot từ CTS engine khi xây dựng tree | Cố định ngay sau khi tree được built | Không thay đổi |
| **Clock Latency** | Giá trị STA live, tính từ extracted parasitics | Cập nhật mỗi khi extraction được chạy | Có, cập nhật sau routing và extraction |

Cả hai *đo cùng một physical path* (từ clock source đến Flip-flop). Sự khác biệt là:
- Insertion Delay = "ảnh chụp tại thời điểm CTS xây dựng tree" — dùng để debug CTS quality.
- Clock Latency = "giá trị hiện tại sau full extraction" — dùng trong STA sign-off.

Khi design tiến đến sign-off, Clock Latency từ extracted parasitics là giá trị dùng để tính setup/hold slack.

---

<a name="4"></a>
## 4. Post-CTS Optimization

### 4.1 Bối cảnh và Mục đích

Post-CTS Optimization là bước tối ưu hóa thực hiện *sau khi* toàn bộ CTS (bao gồm cả Post-Conditioning) hoàn thành. Nó không phải là một phần của `ccopt_design`, mà là bước tiếp theo riêng biệt, thực hiện bằng `opt_design -post_cts`.

**Tại sao cần Post-CTS Optimization nếu đã có Post-Conditioning?**
Post-Conditioning là giai đoạn CTS *nội tại*, tập trung vào sửa DRV và skew ngay sau routing clock net. Post-CTS Optimization là bước tối ưu hóa *data path* (và một phần clock path) với clock latency đã được propagated đầy đủ từ routed clock net. Đây là giai đoạn timing closure thực sự.

### 4.2 Điều kiện vào của Post-CTS Optimization

Tại đây, WNS (Worst Negative Slack) và TNS (Total Negative Slack) **phải ở gần zero** — chỉ còn minor violation. Nếu WNS vẫn còn âm nhiều (ví dụ -2ns hoặc hơn), design cần phải quay lại CTS hoặc thậm chí Placement để resolve root cause.

Số lượng FEP (Failing EndPoints) phải ở mức tối thiểu — một vài điểm, không phải hàng trăm.

### 4.3 Các kỹ thuật trong Post-CTS Optimization

**Setup Recovery:** Mục tiêu làm setup slack $\geq 0$ cho tất cả timing path. Kỹ thuật: buffer insertion trên data path, cell sizing (tăng drive strength để giảm gate delay), useful skew refinement, logic restructuring trên critical path.

**Hold Recovery:** Mục tiêu làm hold slack $\geq 0$ cho tất cả timing path. Kỹ thuật: insert hold buffer (thường là buffer với small drive, tạo delay nhân tạo trên data path). Hold fixing là bước *sau* setup fixing vì hold buffer insertion có thể xấu đi setup.

**Lý do setup trước hold:** Nếu fix hold trước, tool có thể insert nhiều hold buffer vào critical path, làm xấu setup slack của path đó. Do đó thứ tự chuẩn là:
$$\text{fix setup} \rightarrow \text{fix hold} \rightarrow \text{incremental cleanup}$$

**DRV repair:** Tương tự như trong Post-Conditioning nhưng bây giờ trên toàn bộ design.

**Transition improvement:** Giảm slew trên các net vi phạm max transition.

**Capacitance reduction:** Giảm net capacitance trên các net vi phạm max cap.

**Skew refinement:** Điều chỉnh thêm skew trong các skew group còn vi phạm target.

### 4.4 Incremental Optimization

Sau lần setup và hold recovery đầu tiên, công cụ chạy thêm *incremental optimization* để "clean up" mà không làm xáo trộn thiết kế quá nhiều:

$$\text{opt\_design} \xrightarrow{\text{full}} \text{opt\_design} \xrightarrow{\text{incremental setup}} \text{opt\_design} \xrightarrow{\text{incremental hold}} \text{write\_db}$$

Incremental optimization áp dụng các thay đổi nhỏ, cục bộ để cải thiện timing tại các điểm vẫn còn vi phạm, trong khi cố gắng không làm xấu đi những gì đã đạt được ở lần optimization trước.

---

<a name="5"></a>
## 5. CTS Outputs và Quality Review

### 5.1 Các Output của CTS

Sau khi hoàn thành CTS và Post-CTS Optimization, design deliverables bao gồm:

- **Optimized gate-level netlist:** Netlist đã bao gồm các clock buffer, inverter, ICG được insert trong CTS, cùng với hold buffer từ post-CTS optimization.
- **Clock-routed DEF database:** File DEF chứa placement và routing của tất cả cell, đặc biệt là clock net đã được fully routed.
- **Timing reports:** `report_timing`, `time_design -post_cts -setup -hold` — báo cáo setup/hold slack cho tất cả timing path.
- **Clock tree quality reports:** `report_ccopt_clock_trees`, `report_ccopt_skew_groups` — báo cáo skew, insertion delay, transition cho từng clock tree và skew group.
- **Power analysis reports:** `report_power`, `report_power -clock_network` — báo cáo power breakdown theo loại (dynamic, leakage, internal) và theo nhóm (clock, combinational, sequential).

### 5.2 Review and Qualify CTS Results — 4 Nhóm Kiểm tra

Trước khi chuyển sang Routing stage, kết quả CTS phải được review toàn diện theo bốn nhóm:

---

#### Nhóm 1: Timing and Electrical Checks

**Setup và Hold:**
- WNS ≈ 0 (không âm, hoặc chỉ rất nhỏ).
- TNS ≈ 0.
- FEP (Failing EndPoints) ở mức tối thiểu.

**Điều kiện chấp nhận được thực tế:**
Trong hầu hết design flow, sau CTS thường còn một số minor violations. Điều quan trọng là trend phải tốt — WNS và TNS phải *cải thiện đáng kể* so với pre-CTS.

**DRV — Design Rule Violations:**
- Max transition: 0 violation trên cả clock net và signal net.
- Max capacitance: 0 violation.
- Max fanout: 0 violation.
- Max length (nếu có): 0 violation.

---

#### Nhóm 2: Placement and Routability Checks

**Placement Legality:**
CTS có thể di chuyển cell (Cell Relocation, Reconfiguration) và insert cell mới. Sau CTS, tất cả cell phải ở trạng thái *fully legalized*:
- Không overlap (không có hai cell chiếm cùng vị trí).
- Tất cả cell align đúng trên placement row.
- Không có cell unplaced.

**Congestion:**
CTS routing clock net tiêu thụ routing resource. Nếu clock net được route ưu tiên trên upper layers, data routing sẽ phải cạnh tranh trên lower layers. Mục tiêu:
- Routing overflow < 1%: tổng số routing demand vượt quá capacity trên mỗi track không quá 1%.
- Hotspot count < 100: số vùng có congestion cục bộ cao phải trong giới hạn cho phép.

**Placement Density:**
Design density tăng lên sau CTS do clock buffer chiếm thêm diện tích. Density quá cao sẽ gây congestion cho routing sau này. Phải kiểm tra heatmap density.

---

#### Nhóm 3: Power Consumption Checks

**Tại sao CTS ảnh hưởng mạnh đến power:**
Clock network là *major contributor* của dynamic switching power vì:
- Clock signal toggle liên tục ở full clock frequency (không như data signal).
- Clock net có capacitance lớn (dài, rộng, multi-level).
- Mỗi clock edge nạp/phóng điện toàn bộ capacitance của clock net.

$$P_{\text{clock\_dynamic}} = \alpha \cdot C_{\text{clock}} \cdot V_{DD}^2 \cdot f_{\text{clk}}$$

Trong đó $\alpha = 1$ vì clock toggle cả hai edge mỗi cycle, $C_{\text{clock}}$ là tổng capacitance của clock distribution network.

**Ba loại power cần kiểm tra:**
- **Dynamic (switching) power:** Phần lớn đến từ clock network.
- **Leakage power:** Đến từ tất cả cell kể cả clock buffer.
- **Internal power:** Short-circuit current trong quá trình chuyển đổi.

**CTS optimization phải minimize:**
- Số lượng clock buffer không cần thiết (Buffering quá mức).
- Clock switching activity (clock gating là giải pháp — xem ICG concept từ L9).
- Clock tree depth (nhiều level buffer → nhiều switching event).
- Clock capacitance (NDR wire rộng → capacitance lớn → power cao — đây là trade-off của NDR).

---

#### Nhóm 4: Clock Tree Quality Checks

**Clock Skew:**
Phải trong target limit được đặt trong CTS Spec (ví dụ: target_skew = 50ps). Vi phạm skew target không nhất thiết gây timing violation ngay, nhưng báo hiệu rủi ro timing margin thấp và khả năng sign-off failure sau này.

**Clock Insertion Delay:**
Phải được kiểm soát trong target. Insertion delay quá lớn ảnh hưởng đến setup timing budget:

$$\text{Setup Margin} = T_{\text{cycle}} - T_{\text{data\_path}} - T_{\text{insertion\_delay}} - T_{su}$$

Insertion delay lớn → setup margin giảm → design khó meet timing ở high frequency.

**Clock Latency Balance:**
Các sink trong cùng một skew group phải có clock latency gần nhau. Sự mất cân bằng latency giữa các sink cùng skew group là biểu hiện trực tiếp của skew.

**Clock Transition Quality:**
Transition time (slew) của clock signal tại tất cả sink phải thỏa mãn max_tran constraint. Transition quá chậm (slew lớn) sẽ:
- Gây thêm uncertainty vào clock edge timing (slew-induced clock uncertainty).
- Tăng clock-to-Q delay của Flip-flop.
- Tiêu thụ nhiều short-circuit current hơn.

---

### 5.3 CTS Checklist Tổng hợp

| Category | Metric | Target |
|---|---|---|
| **Timing** | Setup WNS | ≥ 0 ns (hoặc gần 0) |
| **Timing** | Hold WNS | ≥ 0 ns |
| **Timing** | TNS | ≈ 0 |
| **Timing** | FEP | Tối thiểu |
| **Electrical** | DRV (max_tran, max_cap, max_fanout) | 0 violation |
| **Placement** | Cell legality | Fully legalized |
| **Routability** | Congestion overflow | < 1% |
| **Routability** | Hotspot count | < 100 |
| **Power** | Total power | Trong design target |
| **Power** | Clock network power | Optimized |
| **Clock Quality** | Clock skew | ≤ target_skew |
| **Clock Quality** | Insertion delay | ≤ target (trong CTS Spec) |
| **Clock Quality** | Clock transition | ≤ max_tran |

---

<a name="6"></a>
## 6. Thiết lập CTS trên Design Mới — Three-Stage Flow

Đây là best practice được Cadence Innovus khuyến nghị cho quá trình *debug và validate* CTS Spec trên một design mới, trước khi commit vào full CTS run. Ba giai đoạn là: **Cluster → Trial → Full**.

Thứ tự này giúp *fail fast* — phát hiện vấn đề sớm ở các giai đoạn rẻ hơn, tránh commit vào full run chỉ để phát hiện vấn đề ở cuối.

---

### 6.1 Cluster Mode

**Mục đích:** Chỉ thực hiện giai đoạn Clustering — nhóm sink và insert buffer để thỏa mãn DRV. **Không có skew balancing. Không có routing.**

**Cài đặt:** `set_db cts_balance_mode cluster` → `ccopt_design`

**Thông tin có được từ Cluster Mode:**
- Số lượng buffer insert vào (indicator về complexity của clock tree).
- Insertion delay ước tính ban đầu (trước balancing và routing).
- **Maximum insertion delay path:** Con đường từ clock source đến sink có insertion delay dài nhất. Đây là chỉ số quan trọng về "worst-case clock latency" của design.

**Các vấn đề phát hiện được từ Cluster Mode:**

Nếu maximum insertion delay path bất thường cao, nguyên nhân thường là:
- **Floorplan blockage:** Blockage ngăn clock net route trực tiếp, buộc phải đi vòng, tăng wire length → tăng delay. Dễ nhận biết trên Clock Tree Layout View: clock path có hình chữ "S" hoặc "Z" thay vì đường thẳng.
- **FIXED clock gates:** ICG cell bị cố định vị trí (FIXED status) ở xa cluster sink → wire dài từ ICG đến Flip-flop.
- **Divider Flip-flop:** Flip-flop dùng để chia tần số được đặt xa clock source nhưng lại là part của critical clock path.
- **Power domain crossings:** Clock đi qua power domain boundary cần thêm level shifter hoặc isolation cell → thêm delay.
- **Uneven cluster distribution:** Một skew group có rất nhiều sink trong khi skew group kia có rất ít → insertion delay không đồng đều giữa groups.

**Ví dụ minh họa (từ Cluster CTD Example trong slide):**
CTD (Clock Tree Debugger) hiển thị bar chart insertion delay per sink. FF6 có insertion delay 490ps, trong khi tất cả FF khác chỉ có ~310–330ps. Tooltip cho biết: "FF6: 490ps — blockage detour adds +168ps". Nhìn vào Clock Tree Layout View, clock path đến FF6 đi vòng quanh một blockage. Giải pháp: điều chỉnh floorplan để remove hoặc move blockage, giảm insertion delay của path đó.

---

### 6.2 Trial Mode

**Mục đích:** Thực hiện full Clustering *cộng với virtual delay balancing*. Thay vì insert real buffer để balance, tool áp dụng *virtual (estimated) delay* vào các node để mô phỏng kết quả sau balancing. **Không insert real balancing buffer. Không routing.**

**Cài đặt:** `set_db cts_balance_mode trial` → `ccopt_design`

**Lợi ích của Trial Mode:**
- Nhanh hơn nhiều so với full run (không routing, không real cell insert).
- Cho phép chạy `time_design -post_cts` trên kết quả virtual để ước tính timing slack **trước khi commit vào full run**.
- Phát hiện **conflicting skew group constraints** sớm.

**Thông tin có được từ Trial Mode:**

CTD hiển thị *simulated arrival time* với virtual delay annotated trên các node bị ảnh hưởng. Bar chart cho thấy arrival time ước tính của từng sink so với balancing target line.

**Ví dụ minh họa (Trial CTD Example):**
Design có hai skew group: SGA và SGB, cùng target 320ps. Sau trial balancing:
- SGA: tất cả sink có arrival time ~315–322ps → gần target → OK.
- SGB: các sink có arrival time 374–378ps → overshoot ~55ps so với target → CONFLICT.

Tooltip: "SGB overshoots by ~55ps — Conflicting skew group constraints". Nguyên nhân: SGB có clock path vật lý dài hơn SGA đáng kể — nếu giữ target 320ps cho cả hai, tool không thể đạt được vì insertion delay tự nhiên của SGB đã vượt 320ps. Giải pháp: hoặc nới lỏng target của SGB, hoặc điều chỉnh floorplan để các sink của SGB gần clock source hơn.

**So sánh với không dùng Trial Mode:**
Nếu bỏ qua trial và chạy thẳng full, vấn đề conflicting skew group constraint chỉ được phát hiện sau khi routing hoàn thành — mất nhiều thời gian hơn nhiều để debug.

---

### 6.3 Full Mode

**Mục đích:** Chạy toàn bộ CTS flow: Clustering + real buffer insertion + Balancing + Routing (NanoRoute) + Post-Conditioning + data path timing optimization đồng thời. Đây là production mode mặc định.

**Cài đặt:** `set_db cts_balance_mode full` → `ccopt_design` (đây cũng là default nếu không set)

**Thông tin có được từ Full CTD:**

CTD hiển thị clock tree với hai loại buffer được phân biệt màu sắc:
- **DRV buffer (green/xanh lá):** Được insert trong Clustering để thỏa mãn DRV (max_tran, max_cap, max_fanout). Không liên quan đến skew.
- **Balancing buffer (yellow/vàng):** Được insert trong Balancing để cân bằng insertion delay. Liên quan trực tiếp đến skew.

Bar chart arrival time sau full run cho thấy tất cả sink có arrival time trong một *skew band* hẹp, xác nhận tree đã balanced.

**Ví dụ minh họa (Full CTD Example):**
Sau full run, tất cả 8 sink (FF0–FF7) có arrival time trong khoảng 319–322ps. Skew = max - min = 322 - 316 = **6ps** — rất nhỏ, đạt target.

Header của CTD xác nhận: "NanoRoute | NDR: 2x width / 2x spacing" — clock net đã được routed với NDR.

**Sau Full Mode:**
Chạy `opt_design -post_cts -setup` → `opt_design -post_cts -hold` → incremental cleanup → `write_db`.

---

<a name="7"></a>
## 7. Clock Tree Debugger (CTD)

CTD (Clock Tree Debugger) là công cụ visualization trong Cadence Innovus, cung cấp các view đồ họa của clock tree đã được synthesize.

### 7.1 Hai View chính của CTD

**Clock Insertion Delay View (Waveform/Tree View):**

View này vẽ clock tree dạng waveform theo chiều dọc với insertion delay trên trục Y. Từ trên xuống dưới:
- **Root (blue):** Clock source ở đỉnh.
- **DRV buffer (green):** Buffer insert để fix DRV — nằm ở các level trên.
- **Balancing buffer (yellow/gold):** Buffer insert để balance skew — nằm ở các level trung gian.
- **ICG cells (green, hình tròn):** Clock gating cell.
- **Flip-flops (red, hình tam giác ngược):** Sink, ở đáy của view.

**Ý nghĩa của phân bố Flip-flop trong view:**
- Các Flip-flop tập trung tightly ở cùng một level Y → skew nhỏ (clock tree balanced tốt).
- Các Flip-flop trải rộng trên nhiều level Y → skew lớn (cần debug).
- Một Flip-flop đơn lẻ nằm rất cao so với nhóm → clock path đến FF đó ngắn bất thường (có thể là exclude pin hoặc ignore pin misconfiguration).

**Clock Tree Layout View:**

View này hiển thị vị trí vật lý của tất cả clock cell và clock routing trên die. Engineers có thể trace clock path từ source đến sink để phát hiện routing detour (do blockage), unusual topology, hoặc NDR vi phạm.

### 7.2 Cross-View Navigation

Đây là tính năng quan trọng nhất của CTD: *selecting a cell or path in one view highlights the corresponding elements in the other view*. Khi click vào một Flip-flop trong Insertion Delay View, Layout View sẽ highlight clock path vật lý đến Flip-flop đó. Ngược lại, click vào một wire trong Layout View sẽ highlight node tương ứng trong tree view.

Điều này cho phép engineers nhanh chóng correlate giữa timing data (insertion delay, skew) và vị trí vật lý — rất hữu ích khi debug outlier paths.

### 7.3 CTD Insights theo từng CTS Mode

| Mode | CTD hiển thị | Thông tin chính |
|---|---|---|
| **Cluster** | DRV buffers, insertion delay per sink | Maximum insertion delay path; outlier sinks |
| **Trial** | DRV buffers + virtual delay annotations | Simulated arrival times vs target; skew group conflict |
| **Full** | DRV (green) + Balancing (yellow) buffers + real arrivals | Actual skew; NDR confirmation; balanced cluster |

---

<a name="8"></a>
## 8. Inverter-Based vs Buffer-Based Clock Trees

### 8.1 Đặc điểm điện tử của Clock Cell

Cả clock inverter và clock buffer đều được *characterized cho balanced rise/fall delay* — điều này **khác biệt hoàn toàn** so với regular functional cell.

Trong regular cell (ví dụ AND gate):
- Rise và fall delay có thể khác nhau đáng kể tùy vào transistor sizing.
- Sự bất đối xứng này không quan trọng với data path (chỉ cần đáp ứng setup/hold tại destination FF).

Trong clock cell:
- Rise delay phải xấp xỉ bằng fall delay.
- **Lý do:** Nếu một clock inverter có rise/fall delay khác nhau, thì hai clock path đi qua số lượng inversions khác nhau (một số chẵn, một số lẻ) sẽ có clock arrival time bị shift khác nhau. Tổng skew giữa các path đó sẽ phụ thuộc vào tích của *số lượng inversion asymmetry* và *rise/fall timing mismatch*. Với clock buffer (two inverter stages), rise/fall mismatch bị tự triệt tiêu do hai inversion.

$$\Delta T_{\text{skew\_from\_asymmetry}} = N_{\text{inversions\_difference}} \times (T_{\text{rise}} - T_{\text{fall}})$$

Nếu $T_{\text{rise}} = T_{\text{fall}}$, mọi path đều có cùng delay bất kể số lần inversion → không có skew từ asymmetry.

### 8.2 Clock Inverter

**Cấu trúc:** Cell một tầng (single-stage) — một PMOS và một NMOS nối tiếp, output là nghịch đảo của input.

**Ưu điểm:**
- Delay per cell thấp hơn clock buffer (ít stage hơn).
- Area nhỏ hơn clock buffer (ít transistor hơn).

**Nhược điểm và ràng buộc:**
- **Polarity constraint:** Mỗi inverter đảo polarity. Mỗi root-to-sink path phải duy trì *số lần inversion đúng* (chẵn hoặc lẻ tùy yêu cầu của clock edge). Nếu một path có 3 inverters và path kia có 4, chúng sẽ nhận clock edge ngược nhau — đây là lỗi chức năng nghiêm trọng.
- **Polarity balancing overhead:** Tool phải đảm bảo mọi path từ root đến mọi sink đều có cùng polarity. Điều này thêm ràng buộc topology và có thể làm phức tạp quá trình balancing.
- **Tree construction khó hơn:** CTS engine phải track polarity qua mỗi node, không chỉ insertion delay.

### 8.3 Clock Buffer

**Cấu trúc:** Thường là hai inverter stages nối tiếp. Stage đầu tiên nhỏ (weak drive), stage thứ hai mạnh hơn (để drive wire capacitance và sink load).

**Ưu điểm:**
- Polarity-preserving: output cùng polarity với input → không cần track và balance polarity.
- Drive strength mạnh hơn clock inverter đơn → có thể drive wire dài hơn mà không vi phạm max transition.
- **Ít stages hơn cho cùng wire length:** Vì buffer có drive strength mạnh hơn, buffer-based tree cần ít stages hơn cho cùng một chiều dài clock path.

**Hệ quả của ít stages:**
$$P_{\text{dynamic}} \propto N_{\text{stages}} \times C_{\text{per\_stage}} \times V_{DD}^2 \times f$$

Ít stages hơn → tổng switching power thấp hơn với cùng clock frequency.

Ít stages hơn → ít accumulated variation → total insertion delay variation nhỏ hơn → skew sensitivity thấp hơn.

### 8.4 Kết luận và Lựa chọn Thực tế

**Buffer-based trees là practical default** cho main clock distribution vì:
- Implementation đơn giản hơn (không cần quản lý polarity).
- Stage count thấp hơn → power và variation tốt hơn.
- Library cell optimization cũng thường hướng đến clock buffer nhiều hơn.

**Inverter-based hoặc mixed trees** được chọn khi:
- **Polarity requirement:** Một số design yêu cầu inverted clock tại một số Flip-flop (negedge triggered FF) — inverter có thể đảm nhận cả vai trò tạo inverted clock lẫn buffer.
- **Pulse-width behavior:** Trong một số kịch bản, inverter-pair có thể preserve duty cycle tốt hơn trong một số process corners.
- **Library-specific optimization:** Một số PDK có clock inverter được characterized tốt hơn clock buffer, cho phép mixed strategy đạt delay/power tốt hơn.

---

<a name="9"></a>
## 9. Virtual Clock và Generated Clock

Đây là hai khái niệm liên quan đến clock definition trong SDC, được đề cập trong phần Backup slides của L10.

### 9.1 Virtual Clock

**Định nghĩa:** Virtual Clock là clock được định nghĩa trong SDC *nhưng không có physical source* trong design — không gắn với bất kỳ clock port hay clock pin nào.

**Đặc điểm:**
- Không tồn tại vật lý trong hardware.
- Không được CTS synthesize thành clock tree (vì không có source pin để bắt đầu build tree).
- Được dùng như *timing reference* cho các interface path: input delay và output delay.

**Mục đích sử dụng:**

Khi design có I/O interface với một chip ngoài, chip ngoài sử dụng một clock riêng (`CLK_CFG`, `CLK_SAD`…) không available bên trong DUT (device under test). Để STA có thể phân tích timing của I/O path (từ input port đến FF, hoặc từ FF đến output port), cần biết clock period và waveform của chip ngoài đó. Virtual Clock đóng vai trò đó.

**Ví dụ (từ slide):**

```
create_clock -name VIRTUAL_CLK_SAD -period 10 -waveform {0 5}
create_clock -name VIRTUAL_CLK_CFG -period 8  -waveform {0 4}
set_input_delay  -clock VIRTUAL_CLK_SAD -max 2.7 [get_ports ROW_IN]
set_output_delay -clock VIRTUAL_CLK_CFG -max 4.5 [get_ports STATE_O]
```

Ý nghĩa: Port `ROW_IN` có data đến 2.7ns trước clock edge của `VIRTUAL_CLK_SAD` (period 10ns) — STA dùng thông tin này để tính setup/hold slack của path từ `ROW_IN` đến Flip-flop bên trong.

**Liên hệ với CTS:** Virtual clock không ảnh hưởng đến CTS vì không có physical source. Tuy nhiên STA sau CTS phải dùng đúng virtual clock definition khi analyze I/O timing paths.

**Lưu ý về latency:** Người thiết kế có thể define custom latency cho virtual clock (thể hiện clock latency của board trace từ nguồn clock bên ngoài đến input pin của chip). Điều này cải thiện độ chính xác của STA cho I/O path.

---

### 9.2 Generated Clock

**Định nghĩa:** Generated Clock là clock *dẫn xuất từ một clock khác* (master clock). Master clock phải có physical source (port hoặc pin).

**Cơ chế tạo ra Generated Clock:**

| Nguồn gốc | Ví dụ thực tế |
|---|---|
| **Frequency divider** | Flip-flop chia đôi tần số: CLKPDIV2 = CLKP / 2 |
| **PLL output** | PLL nhân tần số: CLKX2 = CLK × 2 |
| **Phase shift** | Phase-shifted: CLK_SHIFT = CLK offset 90° |
| **Clock gating cell (ICG)** | Gated clock = CLK AND EN |
| **Multiplexer** | Clock mux để chọn giữa hai clock source |

**Tại sao Generated Clock phải được khai báo tường minh trong SDC:**

Nếu không khai báo `create_generated_clock`, STA tool sẽ tự *trace* clock từ master clock xuyên qua logic gate. Trong một số trường hợp trace này không chính xác (đặc biệt qua sequential logic hoặc mux với phức tạp), dẫn đến:
- STA không nhận ra đây là một *clock domain mới* cần CTS riêng.
- Clock relationship giữa master và generated domain bị phân tích sai.
- CDC (Clock Domain Crossing) paths có thể bị analyze với wrong clock relationship.

Khi khai báo tường minh, STA biết:
- Đây là generated clock, source của nó là master clock.
- Relationship (divide, multiply, phase shift) giữa hai clock là gì.
- Cần build sub-tree bắt đầu từ generated clock source pin.

**Ví dụ (từ slide):**

```
create_clock -name CLKP -period 10 [get_pins UPLL0/CLKOUT]

create_generated_clock -name CLKPDIV2 \
    -source [get_pins UPLL0/CLKOUT] \
    -divide_by 2 \
    [get_pins UFF0/Q]

create_generated_clock -name CLKX2 \
    -source [get_ports CLK] \
    -multiply_by 2 \
    [get_pins UPLL1/CLKOUT]

create_generated_clock -name CLK_SHIFT \
    -source [get_ports CLK] \
    -phase 90 \
    [get_pins UBUF1/O]
```

**Liên hệ với CTS:**

Từ L9 (phần 8), CTS phải build clock tree cho *tất cả* physical clock — cả primary và generated. Generated clock có source point là pin/port khác với primary clock. CTS engine sẽ build sub-tree bắt đầu từ source của generated clock, sử dụng clock period của generated clock (khác với period của master clock) để tính DRV và timing constraints.

**Generated clock từ ICG:** Khi ICG gating một clock để tạo ra *functional gated clock*, output của ICG (CLKOUT) thực chất là một generated clock. Đây chính là lý do trong CTS Spec, ICG output pin được set `sink_type nonstop` — CTS nhìn xuyên qua ICG và tiếp tục build clock tree cho tất cả Flip-flop downstream.

---

<a name="10"></a>
## 10. Mối Liên Kết Tổng Hợp L9 + L10 — Hệ Nhân Quả Hoàn Chỉnh

### 10.1 Chuỗi logic từ Pre-CTS đến Sign-off

```
Pre-CTS (L9)                CTS Execution (L10)           Post-CTS (L10)
─────────────────           ────────────────────────────   ─────────────────
Clock Definition (SDC)  →   Clustering                  →  Post-CTS opt
  primary clock             (grouping + DRV fix)            (setup recovery)
  generated clock           ↓                               (hold recovery)
NDR Rules (CTS Spec)    →   Balancing                   →  (incremental)
Exception Pins              (skew minimization)             ↓
  stop/nonstop/             ↓                              Quality Review
  exclude/ignore/           Routing Clock Tree              (timing checks)
  through                   (NDR, shielding, upper          (DRV checks)
Skew Groups                 metal, multi-cut via)           (placement)
Target Skew/Trans           ↓                               (congestion)
Buffer/Inv Cell List    →   Post-Conditioning           →  (power)
                            (propagated delay,              (clock quality)
                            CPPR, repair violations)        ↓
                                                           Sign-off
```

### 10.2 Cascade của Optimization Techniques

Mỗi kỹ thuật optimization trong mục 2 giải quyết một symptom cụ thể:

| Symptom | Root Cause | Kỹ thuật giải quyết |
|---|---|---|
| Skew cao tại một cluster | Load distribution không đều | Cell Relocation |
| Một FF có insertion delay outlier | Tree depth không phù hợp | Level Adjustment |
| Một nhánh quá tải, nhánh kia quá nhẹ | Phân vùng cluster không tối ưu | Reconfiguration |
| Nhánh ngắn vs nhánh dài trong cùng sub-tree | Asym wire length | Dummy Load Insertion |
| Transition violation tại nhiều Flip-flop | Fanout quá lớn cho một driver | Load Splitting |
| Setup violation còn sót sau CTS | Không đủ setup margin với balanced tree | Useful Skew |
| Insertion delay quá lớn trên critical path | Gate delay trên clock tree | VT-Swapping (Low-VT) |

### 10.3 Mối liên hệ giữa Three-Stage Flow và các giai đoạn nội tại CTS

| Stage | Giai đoạn nội tại thực hiện | Mục đích debug |
|---|---|---|
| **Cluster** | Clustering only | Phát hiện floorplan issues → insertion delay outlier |
| **Trial** | Clustering + virtual balancing | Phát hiện constraint conflicts → arrival time overshoot |
| **Full** | Tất cả 4 giai đoạn | Production result → timing closure |

### 10.4 Mối liên hệ giữa NDR (L9) và Clock Tree Quality (L10)

NDR (từ L9 phần 9) ảnh hưởng trực tiếp đến:
- **Insertion delay:** Wire rộng → resistance thấp → delay thấp hơn; nhưng wire rộng cũng tăng capacitance → delay tăng. Trade-off này được balance tại NDR design rule.
- **Transition quality:** Wire rộng + driver sizing đúng → slew tốt hơn → thỏa mãn max_tran.
- **Clock skew variation:** NDR reduce RC variation → reduce insertion delay variation → reduce skew sensitive đến process/temperature variation.
- **Power:** Clock net với NDR 2x width → capacitance ~ 2x → dynamic power ~ 2x. Đây là cost của NDR.

### 10.5 CPPR và Timing Accuracy

CPPR (từ mục 3.4) giải thích tại sao timing result *sau* CTS (post-CTS với propagated clock) tốt hơn timing result *trước* CTS (ideal clock):

**Trước CTS:** Clock dùng `ideal_clock_latency` — mô hình đơn giản hóa. STA phân tích setup slack với worst-case uncertainty cho tất cả clock paths.

**Sau CTS với propagated clock:** Clock dùng actual propagated delay từ extracted netlist. CPPR loại bỏ pessimism từ common clock path. Kết quả là:
$$\text{Slack}_{\text{post-CTS with CPPR}} > \text{Slack}_{\text{pre-CTS with ideal clock}}$$

Sự cải thiện này không phải vì design "tốt hơn" — mà vì phân tích *chính xác hơn*.

### 10.6 Impact of CTS — Tổng hợp side effects

CTS không chỉ giải quyết clock distribution mà còn tác động đến nhiều khía cạnh khác của design:

**Tăng design density:**
Clock buffer insert thêm cell vào placement area. Với design có 11,537 sinks và 211 clock buffer sau CTS (từ Balancing Log), diện tích tăng thêm. Điều này có thể gây congestion downstream.

**Cell relocation:**
Non-clock cell có thể bị moved ra khỏi vị trí tối ưu để nhường chỗ cho clock cell. Điều này ảnh hưởng đến data path timing và routing.

**Routing resource consumption:**
Clock net routing (NDR + shielding) chiếm routing track không chỉ cho bản thân clock net mà cả cho shield net (VSS wire). Upper layer routing track bị chiếm dụng nhiều hơn.

**Mới phát sinh timing violation:**
Vì non-clock cell có thể bị relocated, data path delay thay đổi → có thể phát sinh mới setup/hold violation không tồn tại trước CTS. Đây là lý do Post-CTS Optimization bước tiếp theo là bắt buộc.

**Power increase:**
Số lượng cell tăng (211 buffer thêm vào) + NDR clock net capacitance lớn → dynamic power tăng. CTS optimization phải balance giữa clock quality (skew, transition) và power budget.

---

*Hết tài liệu — CTS Part II (L10). Toàn bộ nội dung tích hợp từ Lecture 10 (Tresemi) và cross-referenced với Lecture 9.*
