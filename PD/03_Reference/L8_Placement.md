# Lecture 8 — Placement (Part II): Post-Placement Optimization

### Phân tích lý thuyết toàn diện (kết hợp tham chiếu từ L7)

---

## 1. Tổng quan vị trí trong Design Flow

L8 là phần tiếp nối trực tiếp của L7. Nhìn lại toàn bộ flow Placement, có thể chia thành ba giai đoạn lớn:

- **Pre-Placement** (L7): Sanity checks — kiểm tra tính hợp lệ của Floorplan, DRC shorts, power domain, PG nets trước khi chạy Placement.
- **Placement** (L7): Global placement → Detail placement → High Fanout Net Synthesis (HFNS) → Scan chain reordering.
- **Post-Placement** (L8 — trọng tâm của bài này): DRV fixing (đã giới thiệu cuối L7) → PreCTS Timing optimization → **Power & Area optimization** → **Tie-cell insertion** → **Spare cell insertion** → **MBFF optimization** → **Congestion analysis** → **Guides to minimize congestion**.

Điểm cốt lõi cần hiểu ngay từ đầu: **Post-Placement là một vòng lặp lặp đi lặp lại (iteration loop)**, không phải một chuỗi tuyến tính. Sau mỗi bước tối ưu, kết quả được đánh giá lại (Timing, Congestion, Power), và nếu chưa đạt yêu cầu, engineer quay lại điều chỉnh chiến lược rồi chạy lại. Điều kiện thoát khỏi vòng lặp là toàn bộ các tiêu chí exit đều được đáp ứng: không DRV, không EMIR, PPA đạt yêu cầu, tất cả Cell được legalize, congestion nằm trong ngưỡng chấp nhận được.

---

## 2. Power Analysis — Nền Tảng Lý Thuyết

### 2.1 Hai thành phần của tổng công suất

Mọi mạch số CMOS đều tiêu thụ công suất theo hai cơ chế độc lập nhau về bản chất vật lý:

$$P_{\text{total}} = P_{\text{static}} + P_{\text{dynamic}}$$

**Static Power (Leakage Power)**: Phát sinh ngay cả khi không có switching nào xảy ra — tức là transistor đang ở trạng thái "lý tưởng OFF" nhưng vẫn cho dòng chạy qua do các hiệu ứng vật lý ở cấp độ bán dẫn. Static power phụ thuộc vào loại Cell (HVT/SVT/LVT), nhiệt độ, process corner, và trạng thái logic của mạch.

**Dynamic Power**: Phát sinh khi có switching (trạng thái logic thay đổi), bao gồm hai thành phần con: Switching Power và Internal Power.

### 2.2 Dynamic Power — phân tích chi tiết

#### Switching Power (thành phần dominant)

Cơ chế vật lý: Mỗi lần output của một Standard Cell chuyển từ '0' sang '1', tụ điện tải $C$ phải nạp từ GND lên VDD (dòng $I_{\text{charge}}$). Khi chuyển từ '1' về '0', tụ xả qua transistor NMOS xuống GND (dòng $I_{\text{discharge}}$). Năng lượng nạp vào tụ trong mỗi chu kỳ là $\frac{1}{2}CV^2$, nhưng mỗi chu kỳ clock có cả nạp lẫn xả, nên:

$$P_{\text{switching}} = \alpha \cdot C \cdot V^2 \cdot f$$

Trong đó:

- $\alpha$: Switching activity factor ($0 < \alpha \leq 1$) — xác suất có một cạnh chuyển (0→1 hoặc 1→0) xảy ra trong một chu kỳ clock. $\alpha = 0.1$ nghĩa là trung bình 10% số chu kỳ có switching.
- $C$: Load capacitance — bao gồm capacitance của dây kết nối (wire capacitance), input capacitance của các Cell nhận tín hiệu (fanout), và output capacitance của Cell driver.
- $V$: Supply voltage — công suất tỉ lệ bậc hai với $V$, đây là lý do giảm voltage là kỹ thuật tiết kiệm điện hiệu quả nhất.
- $f$: Clock frequency — công suất tỉ lệ tuyến tính với tần số.

**Hệ quả trực tiếp cho Placement**: $C$ tỉ lệ với wirelength (độ dài wire). Do đó, việc đặt các Cell có kết nối logic gần nhau nhau về mặt vật lý sẽ **giảm wire capacitance → giảm $P_{\text{switching}}$**. Đây là nguyên lý cơ bản của wirelength-driven placement optimization.

#### Internal Power

Phát sinh bên trong Standard Cell, bao gồm hai thành phần:

**Short-circuit Power**: Trong khoảng thời gian ngắn $t_{\text{sc}}$ khi input đang chuyển trạng thái (không phải '0' cứng hay '1' cứng mà nằm ở mức trung gian), cả transistor PMOS và NMOS có thể **đồng thời dẫn điện**, tạo ra đường dẫn điện thẳng từ VDD xuống GND. Dòng ngắn mạch $I_{sc}$ này không làm được việc gì có ích:

$$P_{\text{short_circuit}} \approx I_{sc} \cdot V \cdot t_{sc} \cdot f$$

Short-circuit power tỉ lệ với thời gian chuyển tiếp tín hiệu $t_{sc}$, tức là với **Transition time (Slew)**. Khi Slew quá lớn, transistor ở mức trung gian lâu hơn → short-circuit power tăng. Đây là lý do tại sao DRV max_transition không chỉ ảnh hưởng đến Timing mà còn ảnh hưởng đến Power.

**Internal Node Switching**: Các node nội bộ bên trong Cell cũng có capacitance và có thể switching, đóng góp một lượng nhỏ vào Internal Power.

### 2.3 Static Power (Leakage) — phân tích chi tiết

$$P_{\text{leakage}} = V \cdot I_{\text{leak}}$$

Trong đó $I_{\text{leak}}$ là tổng của nhiều dòng rò rỉ:

- $I_1$: Diode reverse bias current
- $I_2$: Sub-threshold current (thành phần lớn nhất ở advanced nodes)
- $I_3$: Gate-induced drain leakage (GIDL)
- $I_4$: Gate oxide leakage

Ba thành phần chính của leakage được phân tích như sau:

#### Gate Leakage Current

- **Cơ chế**: Hiện tượng tunneling lượng tử qua lớp gate oxide mỏng. Khi điện trường qua oxide đủ mạnh, electron "xuyên hầm" (tunnel) qua lớp oxide, tạo dòng chảy từ gate electrode vào kênh dẫn (hoặc ngược lại).
- **Điều kiện**: Xảy ra ngay cả khi transistor đang "OFF" hoàn toàn (không phụ thuộc vào $V_{gs}$).
- **Xu hướng theo technology scaling**: Gate oxide ngày càng mỏng hơn theo mỗi node công nghệ mới → tunneling probability tăng → Gate leakage tăng mạnh. Đây là động lực chuyển sang high-k dielectric material (ví dụ: HfO2 thay SiO2) ở các advanced nodes.

#### Subthreshold Leakage Current — thành phần dominant

- **Cơ chế**: Lý tưởng, khi $V_{gs} < V_{th}$, MOSFET không dẫn điện. Thực tế, dòng điện không bị cắt ngay lập tức mà giảm theo hàm mũ khi $V_{gs}$ giảm dưới $V_{th}$. Cơ chế này gọi là **subthreshold conduction** — carrier vẫn có thể khuếch tán từ source sang drain nhờ gradient nồng độ, dù không có drift field.
- **Công thức đặc trưng**: $I_{\text{sub}} \propto e^{V_{gs}/nV_T}$ với $n$ là subthreshold swing factor, $V_T = kT/q$ là thermal voltage.
- **Tầm quan trọng**: Là thành phần leakage chiếm tỷ trọng lớn nhất ở các technology nodes nhỏ hơn 65nm.
- **Hệ quả thiết kế**: Giảm $V_{th}$ → tăng performance (transistor ON nhanh hơn) nhưng đồng thời tăng subthreshold leakage theo cấp số nhân. Đây là **fundamental tradeoff** giữa performance và leakage power.

#### Reverse-Biased Junction Leakage

- **Cơ chế**: Tại các tiếp giáp PN giữa source/drain và substrate, khi tiếp giáp được phân cực ngược (reverse bias), tồn tại dòng rò nhỏ do generation-recombination của carrier trong vùng depletion.
- **Điều kiện xuất hiện**: Với NMOS: source/drain (vùng N+) ở điện thế cao hơn substrate (P) → junction reverse biased. Với PMOS: source/drain (vùng P+) nối VSS khi PMOS OFF → junction reverse biased.
- **Đặc tính**: Thành phần này tương đối nhỏ và ít thay đổi theo technology scaling so với subthreshold và gate leakage.

### 2.4 Xu hướng Leakage theo Technology Scaling

Đây là một trong những thách thức lớn nhất của semiconductor industry từ khoảng node 130nm trở đi:

|Technology Node|Đặc điểm|
|---|---|
|250nm – 130nm|Dynamic power chiếm ưu thế, leakage nhỏ, có thể bỏ qua|
|90nm – 65nm|Leakage bắt đầu đáng kể, bằng ~10-20% dynamic|
|45nm – 28nm|Leakage tăng vọt, bắt đầu ngang bằng dynamic power|
|22nm – 16nm|Leakage có thể vượt dynamic power, trở thành dominant|

**Lý do leakage tăng khi scaling**:

1. Supply voltage giảm → buộc phải giảm $V_{th}$ để duy trì ON-current (performance) → subthreshold leakage tăng theo cấp số nhân.
2. Gate oxide mỏng hơn (từ ~10nm ở 250nm node xuống ~1.2nm ở 45nm node) → gate tunneling tăng mạnh.
3. Short-channel effects trở nên nổi bật: Drain-Induced Barrier Lowering (DIBL) làm giảm $V_{th}$ hiệu dụng khi $V_{ds}$ tăng, từ đó tăng leakage.

**Ý nghĩa cho PD flow**: Ở các advanced nodes (28nm và nhỏ hơn), leakage optimization không còn là tùy chọn mà là **bắt buộc**. Cả hai loại power đều phải được optimize đồng thời.

---

## 3. Các Yếu Tố Ảnh Hưởng đến Power

### 3.1 Các yếu tố ảnh hưởng đến Leakage Power

|Yếu tố|Ảnh hưởng|Giải thích|
|---|---|---|
|↓ Threshold Voltage ($V_{th}$)|↑ Subthreshold leakage|Giảm $V_{th}$ theo hàm mũ|
|↑ Supply Voltage ($V_{DD}$)|↑ Gate & junction leakage|Điện trường qua oxide mạnh hơn|
|↑ Temperature|↑ Tất cả các thành phần leakage|$V_T = kT/q$ tăng → subthreshold leakage tăng|
|↑ Transistor Width ($W$)|↑ Leakage tỉ lệ $W$|Cell lớn hơn, nhiều current path hơn|
|↓ Oxide Thickness ($t_{ox}$)|↑ Gate oxide tunneling|Tunneling probability tăng theo cấp số mũ|
|Reverse Body Bias (RBB)|↓ Leakage|Tăng $V_{th}$ hiệu dụng → giảm subthreshold leakage|
|Forward Body Bias (FBB)|↑ Leakage|Giảm $V_{th}$ hiệu dụng|
|Process Variation|Biến thiên leakage theo từng device|Fast corner → leakage cao hơn slow corner|
|Logic State (Stacking Effect)|↓ Leakage khi stacking|Nhiều transistor nối tiếp OFF → leakage giảm theo hàm mũ|

**Stacking Effect** đáng chú ý thêm: Khi hai transistor NMOS nối tiếp cùng OFF (như trong cổng NAND 2-input với cả hai input = '0'), điện thế tại node trung gian dâng lên do leakage từ transistor trên, tạo ra reverse body bias tự nhiên cho transistor dưới → $V_{th}$ của transistor dưới tăng → leakage tổng giảm đáng kể so với một transistor đơn lẻ OFF.

### 3.2 Các yếu tố ảnh hưởng đến Dynamic Power

|Yếu tố|Tác động|Cơ chế|
|---|---|---|
|↑ Switching Activity ($\alpha$)|↑ Dynamic power tuyến tính|Nhiều chuyển tiếp hơn trong một giây|
|↑ Capacitance ($C$)|↑ Dynamic power tuyến tính|Nhiều năng lượng hơn để nạp/xả|
|↑ Supply Voltage ($V$)|↑ Dynamic power bậc hai ($V^2$)|Quan trọng nhất — giảm $V$ hiệu quả nhất|
|↑ Frequency ($f$)|↑ Dynamic power tuyến tính|Nhiều chuyển tiếp mỗi giây|
|↑ Fanout / Wirelength|↑ Capacitance → ↑ Dynamic power|Wire dài hơn, nhiều input Cap hơn|
|↑ Glitching / Hazards|↑ Unnecessary transitions|Chuyển tiếp không mang thông tin hữu ích|
|↑ Parallelism|↑ Instantaneous current spikes|Nhiều mạch cùng switching một lúc|

**Glitching** đáng chú ý thêm: Trong combinational logic, do sự chênh lệch delay trên các đường khác nhau đến một cổng logic, output có thể chuyển trạng thái sai (glitch) trước khi đạt giá trị đúng. Những chuyển tiếp thừa này tiêu tốn dynamic power mà không đóng góp vào tính toán. Glitch reduction là một kỹ thuật tối ưu power ở cấp synthesis.

---

## 4. Power Optimization trong Post-Placement

### 4.1 Nguyên lý tổng quát

Khi Power Optimization được kích hoạt (lệnh `opt_power -pre_cts`), công cụ EDA thực hiện một loạt các biến đổi **mà không vi phạm Timing constraints**. Nguyên tắc then chốt: **timing phải được giữ nguyên hoặc cải thiện** — không bao giờ hy sinh timing để lấy power savings trừ khi có Slack dương đủ lớn.

Các hành động cụ thể:

**Leakage reduction:**

- **Cell swapping sang lower-leakage equivalent**: Thay thế Cell có $V_{th}$ thấp (LVT) bằng Cell có $V_{th}$ cao hơn (SVT hoặc HVT) trên các path có Slack dương. LVT nhanh nhưng leakage cao nhất; HVT chậm nhất nhưng leakage thấp nhất.
- **Cell downsizing**: Thay Cell drive strength lớn (ví dụ INV8X) bằng Cell nhỏ hơn (INV4X, INV2X) khi fanout/load không cần drive strength lớn. Cell nhỏ hơn → diện tích transistor nhỏ hơn → leakage nhỏ hơn.

**Dynamic power reduction:**

- **Buffer/Inverter removal**: Xóa Buffer hoặc cặp Inverter đôi (double inverter) không cần thiết — giảm số lượng Cell switching và giảm total capacitance.
- **Wirelength optimization**: Điều chỉnh vị trí Cell để rút ngắn wire → giảm wire capacitance → giảm $P_{\text{switching}}$.

**Constraint tuân thủ**: Toàn bộ quá trình phải tuân thủ UPF (Unified Power Format) power budgets nếu design sử dụng multi-voltage domain.

### 4.2 Thiết lập switching activity để tính dynamic power chính xác

**Vấn đề cốt lõi**: Công thức $P_{\text{switching}} = \alpha \cdot C \cdot V^2 \cdot f$ đòi hỏi biết $\alpha$ — switching activity — cho từng Net. Tuy nhiên, tại giai đoạn Placement, chưa có simulation đầy đủ.

Có ba cách cung cấp switching activity cho tool:

1. **Default assumption**: Đặt global activity mặc định, ví dụ: tất cả combinational node switching 20% số chu kỳ ($\alpha = 0.2$), sequential node (Flip-flop output) switching 80% số chu kỳ. Đây là ước lượng thô nhưng đủ dùng cho giai đoạn Placement.
    
2. **FSDB (Fast Signal Database)**: File nhị phân từ simulation VCS/ModelSim chứa dạng sóng tín hiệu. Tool đọc FSDB và tính $\alpha$ thực tế cho từng Net từ dữ liệu simulation.
    
3. **SAIF (Switching Activity Interchange Format)**: File text chứa pre-computed switching activity statistics. Nhẹ hơn FSDB, dễ xử lý hơn.
    

**Tầm quan trọng**: Accuracy của power analysis tại Placement thấp hơn Post-Route vì: (1) switching activity ở trên chỉ là ước lượng, (2) wire capacitance tại Placement được ước lượng từ wire length, chưa có actual routing. Post-Route power analysis chính xác hơn vì sử dụng simulation-based switching activity (SAIF/FSDB) và extracted parasitics thực tế từ DEF đã routing.

### 4.3 Kết hợp leakage và dynamic trong một hàm tối ưu

Trong thực tế, target của power optimization không phải chỉ minimize leakage hoặc chỉ minimize dynamic, mà là minimize một **weighted combination**:

$$P_{\text{cost}} = w_{\text{leakage}} \cdot P_{\text{leakage}} + w_{\text{dynamic}} \cdot P_{\text{dynamic}}$$

Tham số `opt_leakage_to_dynamic_ratio` (ví dụ = 0.5) điều chỉnh tỷ lệ trọng số: giá trị 0.5 nghĩa là weight bằng nhau. Nếu design là battery-powered (IoT, wearable) → tăng weight của leakage. Nếu design là high-performance processor → leakage ít quan trọng hơn, dynamic power chiếm ưu thế.

---

## 5. Standard Cell VT Selection — Lý thuyết Multi-VT

### 5.1 Ba tier VT cells

Trong một Standard Cell Library hiện đại, mỗi loại Cell logic (AND, OR, Flip-flop, v.v.) thường có **ít nhất ba phiên bản khác nhau về threshold voltage**:

**Low VT (LVT)**:

- $V_{th}$ thấp nhất trong ba tier
- Transistor turn ON nhanh nhất → propagation delay nhỏ nhất → **hiệu năng cao nhất**
- Subthreshold leakage cao nhất vì transistor khó bị "cắt" hoàn toàn
- Sử dụng trên **critical path** để đảm bảo Timing

**Standard/Regular VT (SVT)**:

- $V_{th}$ ở mức trung bình
- Delay và leakage đều ở mức trung bình
- Cell "mặc định" — phần lớn design dùng SVT làm baseline

**High VT (HVT)**:

- $V_{th}$ cao nhất
- Transistor turn ON chậm nhất → delay lớn nhất → **hiệu năng thấp nhất**
- Subthreshold leakage thấp nhất vì transistor được "cắt" mạnh hơn
- Sử dụng trên **non-critical path** để tiết kiệm leakage mà không làm chậm design

### 5.2 Đặc tính quan trọng: Same footprint

Tất cả các phiên bản LVT/SVT/HVT của **cùng một loại Cell** có **kích thước vật lý giống hệt nhau** (same footprint): chiều rộng, chiều cao, và vị trí tất cả các Pin đều giống nhau. Điều này có nghĩa là:

- **Không cần re-Placement** khi swap VT: Tool chỉ thay đổi cell type trong database, vị trí x,y không đổi.
- **Không ảnh hưởng đến Routing**: Tất cả Pin ở cùng vị trí → wire không cần thay đổi.
- Sự khác biệt chỉ ở cấp độ **implant doping level** trong silicon, không ảnh hưởng đến layout vật lý nhìn từ bên ngoài.

### 5.3 Multi-VT optimization strategy

Chiến lược tối ưu power dùng multi-VT là quá trình iterative:

1. **Baseline**: Synthesis cho ra design dùng SVT (hoặc LVT để đảm bảo Timing).
2. **VT Swap - HVT insertion**: Trên các Net/Path có **positive Slack** (dư thừa Timing), swap từ LVT/SVT → HVT. Slack dương cho phép chịu được delay tăng thêm mà không vi phạm Timing.
3. **Re-check Timing**: Đảm bảo không có new Setup violation.
4. **Iterate**: Tiếp tục swap cho đến khi không còn opportunity nào mà không vi phạm Timing.

**Ví dụ định lượng** từ slide: Giả sử một Flip-flop:

- LVT: delay = 1ns, power = 7nW
- SVT: delay = 2ns, power = 6nW
- HVT: delay = 3ns, power = 4nW

Nếu path này có Setup Slack = +2ns, ta có thể swap từ LVT sang HVT (delay tăng 2ns, nhưng Slack giảm về 0, vẫn đạt Timing) và tiết kiệm được 3nW leakage trên Cell này.

---

## 6. Clock Gating — Kỹ Thuật Giảm Dynamic Power Cho Sequential Logic

### 6.1 Vấn đề với Flip-flop khi không có Clock Gating

Flip-flop tiêu thụ dynamic power trên **clock input** ngay cả khi giá trị data D không thay đổi — tức là Flip-flop sẽ "capture" cùng một giá trị mà nó đang lưu trữ. Việc clock vẫn toggle → internal node của Flip-flop vẫn switching → dynamic power bị lãng phí.

Về lý thuyết, một design có 100,000 Flip-flop mà chỉ 30% trong số đó thực sự cần update trong một clock cycle nhất định, 70% còn lại đang tiêu thụ power vô ích chỉ để "capture" cùng giá trị cũ.

### 6.2 Traditional Clock Gating (AND gate approach)

Cách đơn giản nhất: Chèn một cổng AND giữa clock source và clock input của Flip-flop:

```
CLK ──┐
      ├── AND ── CLK_gated ──► Flip-flop
EN ───┘
```

Khi EN = 0, CLK_gated = 0, Flip-flop không nhận clock → không switching → không tiêu thụ dynamic power.

**Vấn đề nghiêm trọng**: Nếu EN thay đổi khi CLK = 1 (EN changes during high phase of clock), output của cổng AND sẽ tạo ra một **glitch** — một clock pulse ngắn không hoàn chỉnh. Glitch này có thể gây ra:

- **False capture**: Flip-flop capture data sai vào thời điểm sai.
- **Setup/Hold violation**.
- **Increased switching activity** paradoxically.

### 6.3 Integrated Clock Gating (ICG) Cell

ICG Cell là giải pháp chuẩn trong modern design. Cấu trúc bên trong gồm:

- Một **Latch** (negative edge-triggered, active low): Latch EN signal vào cuối phase LOW của clock.
- Một **AND gate** hoặc **OR gate** để gate clock.

**Nguyên lý hoạt động**:

```
EN ──────────────► D ──[ Latch ]──► Q ──┐
                    ↑                    ├── AND ──► CLK_gated
CLK (negative) ─────┘                   │
CLK ────────────────────────────────────┘
```

- Khi CLK = 0 (low phase): Latch **transparent** → Q theo EN.
- Khi CLK = 1 (high phase): Latch **opaque** → Q bị giữ nguyên tại giá trị EN đã được capture ở cuối phase CLK = 0.
- Do Q không thay đổi trong khi CLK = 1, CLK_gated = CLK AND Q sẽ là một clock pulse đầy đủ (không glitch) hoặc bằng 0 (hoàn toàn bị gate).

**Kết quả**: CLK_gated chỉ có thể chuyển edge khi CLK = 0, tức là EN chỉ có thể ảnh hưởng đến CLK_gated trong khoảng CLK = 0 — hoàn toàn loại bỏ glitch.

### 6.4 Tại sao ICG Cell quan trọng trong Post-Placement

ICG Cell (ký hiệu `icg_reg_*` trong code) được đặt với thuộc tính `set_dont_touch` trong khi chạy Placement, nghĩa là tool không được di chuyển hoặc swap chúng. Lý do:

- ICG Cell đã được synthesis xác định vị trí và kết nối clock một cách cẩn thận.
- Di chuyển ICG Cell có thể thay đổi clock path length → ảnh hưởng đến Skew.
- ICG Cell phải được CTS (Clock Tree Synthesis) xử lý đặc biệt sau Placement.

---

## 7. Area Optimization trong Post-Placement

### 7.1 Mục tiêu và cơ chế

Area Optimization xảy ra **tự động sau Timing Optimization** hoặc có thể kích hoạt tường minh. Tool thực hiện hai hành động chính, đều nhằm **giảm số lượng Cell và diện tích** mà **không tạo ra Timing violation mới hoặc DRV violation mới**:

**Cell Downsizing**:

- Thay Cell drive strength lớn (ví dụ INV8X) bằng Cell nhỏ hơn (INV4X).
- Nguyên lý: Nếu một Net có fanout thấp, không cần drive strength lớn. Cell drive strength lớn chiếm diện tích silicon lớn hơn (transistor rộng hơn) mà không mang lại benefit về timing.
- Điều kiện an toàn: Chỉ downsize trên Net có **positive Slack** — tức là có "margin" cho phép delay tăng thêm một chút.

**Buffer/Inverter Removal**:

- Loại bỏ các Buffer hoặc cặp Inverter đôi (double-Inverter = inverter nối tiếp inverter, net logic effect = unity buffer) không cần thiết.
- Những Cell này có thể được synthesis chèn vào để fix Timing (buffering net dài) hoặc để đảm bảo DRV (drive strength không đủ), nhưng sau Placement, wirelength đã thay đổi — một số Buffer có thể không còn cần thiết.
- Điều kiện an toàn: Chỉ remove khi không tạo ra max_cap hoặc max_transition violation.

### 7.2 Mối liên hệ giữa Area và Power

Cell downsizing và buffer removal không chỉ giảm area mà còn có **side effect** là:

- Giảm leakage (ít transistor hơn, transistor nhỏ hơn).
- Giảm dynamic power (ít switching node hơn, gate capacitance nhỏ hơn).

Đây là lý do Power Optimization và Area Optimization thường được thực hiện cùng nhau trong một pass `opt_power` hoặc `opt_design`.

---

## 8. Tie Cells — Bảo Vệ Gate Oxide

### 8.1 Vấn đề: Logic '0' và '1' cứng trong Netlist

Trong thiết kế số, thường có input của các Cell được kết nối với logic constant: `1'b0` (logic LOW cứng) hoặc `1'b1` (logic HIGH cứng). Ví dụ: Flip-flop có Scan Enable (SE) luôn bằng 0 khi không ở Scan mode.

Cách naive nhất: Nối trực tiếp Pin đó với VSS (cho `1'b0`) hoặc VDD (cho `1'b1`).

**Vấn đề vật lý nghiêm trọng**: Gate oxide của transistor (lớp SiO2 hoặc high-k dielectric nằm giữa gate electrode và kênh dẫn) là phần **mỏng nhất** và **nhạy cảm nhất** trong transistor hiện đại. Lớp này có thể bị phá hủy vĩnh viễn (oxide breakdown) bởi:

1. **Antenna Effect trong quá trình fabrication**: Trong quá trình ion etching hoặc plasma etching, các dây kim loại dài tích lũy điện tích. Nếu một dây dài kết nối trực tiếp đến gate của transistor mà chưa có path thoát điện tích, điện áp tích lũy có thể lên đến hàng trăm volt → phá hủy oxide.
    
2. **Voltage surge trong operation**: Sự kiện ESD (Electrostatic Discharge) hoặc noise spike có thể tạo ra điện áp vượt mức chịu đựng của oxide.
    

Nếu Pin gate được kết nối trực tiếp với VDD/VSS (nguồn điện lớn, impedance thấp), không có cơ chế nào giới hạn dòng điện khi có surge → oxide bị damage.

### 8.2 Giải pháp: Diode-connected transistor (Tie Cell)

Tie Cell là Standard Cell đặc biệt chứa một transistor được kết nối theo kiểu **diode** (gate và drain nối với nhau):

$$V_g = V_d \implies V_{gs} = V_{ds} \implies V_{ds} > V_{gs} - V_t$$

Điều kiện này đảm bảo transistor **luôn ở saturation region** — hoạt động như một diode. Đặc tính quan trọng của diode-connected transistor là:

- Khi điện áp bình thường: output = VDD (cho Tie_high) hoặc GND (cho Tie_low).
- Khi có voltage surge từ phía gate của Cell nhận: transistor trong Tie Cell **clamp** điện áp, giới hạn dòng điện, bảo vệ oxide của Cell tiếp theo.
- Cung cấp **low-impedance path** để thoát điện tích tích lũy trong Antenna Effect.

### 8.3 Hai loại Tie Cell

**Tie_high Cell (TIEHI)**: Output của Tie_high = logic '1' (VDD). Cấu trúc bên trong: PMOS diode-connected kết nối với VDD, output → logic HIGH. Thay thế mọi kết nối `1'b1` trong Netlist.

**Tie_low Cell (TIELO)**: Output của Tie_low = logic '0' (VSS). Cấu trúc bên trong: NMOS diode-connected kết nối với VSS, output → logic LOW. Thay thế mọi kết nối `1'b0` trong Netlist.

### 8.4 Quá trình Tie Cell Insertion

**Trước khi insert** (Netlist từ synthesis): Chứa các kết nối literal `.D(1'b0)` hoặc `.IN(1'b1)`.

**Sau khi insert** (bởi `add_tieoffs`):

- Tất cả `1'b0` → thay bằng output của TIELO Cell với wire trung gian.
- Tất cả `1'b1` → thay bằng output của TIEHI Cell với wire trung gian.
- Netlist không còn literal constant — mọi thứ đều là Net thực sự.

**Constraints của Tie Cell**:

- **Max fanout**: Một Tie Cell không nên drive quá nhiều Cell receiver (ví dụ tối đa 10 fanout). Lý do: Tie Cell là Cell nhỏ, drive strength thấp; quá nhiều fanout → transition time (Slew) lớn → timing và power bị ảnh hưởng.
- **Max distance**: Tie Cell phải được đặt gần Cell nhận (ví dụ trong vòng 20 micron). Lý do: Wire dài từ Tie Cell đến receiver tạo ra wire capacitance lớn và tăng antenna ratio, đồng thời làm tăng delay.

Sau khi insert, cần chạy `check_place` và `verify_connectivity` để đảm bảo Placement vẫn hợp lệ và Netlist connectivity đúng.

---

## 9. Spare Cells — Cơ Sở Hạ Tầng Cho ECO

### 9.1 Khái niệm và mục đích

Spare Cells (còn gọi là filler logic) là các Standard Cell được chèn vào layout **không phục vụ bất kỳ chức năng logic nào hiện tại** mà được **dự trữ cho các thay đổi thiết kế trong tương lai** — cụ thể là ECO (Engineering Change Order).

**ECO là gì**: Sau khi chip đã tapeout và thậm chí đã fabricate xong (hoặc chuẩn bị tapeout), phát hiện ra bug logic hoặc cần thêm tính năng nhỏ. Thay vì thiết kế lại từ đầu (rất tốn kém), engineer thực hiện ECO: thay đổi kết nối dây ở một số metal layer để "reprogram" logic sẵn có.

**Tại sao cần Spare Cells**: Nếu không có Spare Cells, ECO chỉ có thể tái sử dụng các Cell đang tồn tại bằng cách thay đổi connectivity. Nhưng nếu cần thêm logic gate mới (ví dụ thêm một cổng AND để fix bug), không có chỗ trống để đặt. Spare Cells cung cấp các Cell đã đặt sẵn, đã legalize, đã có power connection — chỉ cần "activate" chúng bằng cách thay đổi kết nối Metal.

### 9.2 Đặc điểm của Spare Cells

**Thành phần**: Thường là mix của:

- Combinational cells: Inverter (INV), NAND, NOR, AND, OR, Buffer
- Sequential cells: Flip-flop (DFFHQX1)
- Chọn loại phản ánh loại logic hay gặp trong ECO: Inverter và NAND là phổ biến nhất vì chúng đủ để implement bất kỳ logic nào (functionally complete).

**Trạng thái khi chèn**:

- **Inputs**: Tất cả inputs của Spare Cells được kết nối với VDD hoặc VSS thông qua Tie Cells (đây là lý do Tie Cell insertion phải xảy ra trước hoặc cùng lúc với Spare Cell insertion). Input không được để floating (lơ lửng) vì floating input có thể oscilate, gây ra switching power không kiểm soát.
- **Outputs**: Tất cả outputs được để **unconnected** (open circuit). Không nối với bất kỳ Net nào.
- **Power**: Tất cả Spare Cells đều có VDD và VSS connections thông qua power rail của Standard Cell row — giống như mọi Cell khác.

**Phân phối trong layout**: Spare Cells phải được **phân tán đều** khắp die, không tập trung ở một góc. Nếu Spare Cells tập trung, ECO chỉ có thể sửa được bug gần khu vực đó; bug ở phần khác của die không thể fix vì quá xa. Thông số `step_x` và `step_y` trong lệnh placement xác định khoảng cách đều đặn giữa các module Spare Cell.

### 9.3 Tại sao Spare Cells là chiến lược kinh tế

Tapeout một chip: chi phí mask set cho advanced node (7nm, 5nm) có thể lên đến **vài triệu USD**. Nếu phát hiện bug sau tapeout, không có Spare Cells → phải re-spin (tapeout lại) → thêm vài triệu USD và vài tháng thời gian.

Chi phí Spare Cells: ~1-5% diện tích die, tương đương vài ngàn USD die cost tăng thêm.

ROI rõ ràng: nếu có bất kỳ ECO nào sau này, Spare Cells đã "trả cho chính nó" rất nhiều lần.

---

## 10. MBFF Optimization — Multi-Bit Flip-Flop

### 10.1 Vấn đề với Single-Bit Flip-Flop

Mỗi Flip-flop đơn lẻ có các đặc điểm:

- Hai Latch (Master và Slave) với internal clock inverters riêng biệt.
- Clock input với buffer riêng.
- Transistors cho data path (D→Q) riêng.

Khi có hàng nghìn Flip-flop trong design, mỗi cái có clock buffer riêng, tổng số buffer và wire length của clock tree rất lớn → clock power cao, clock Skew khó kiểm soát.

### 10.2 MBFF: Cơ chế "banking" Flip-flop

MBFF optimization "gom" (bank) nhiều Single-bit Flip-flop thành một Multi-bit Flip-flop. Ví dụ:

- 2 × 1-bit FF → 1 × 2-bit MBFF
- 4 × 1-bit FF → 1 × 4-bit MBFF
- 8 × 1-bit FF → 1 × 8-bit MBFF

**Điều kiện để bank được**: Các Flip-flop phải có **cùng timing constraints** (cùng clock domain, cùng Setup/Hold constraints). Tool tự động kiểm tra điều kiện này và copy constraints từ các FF đơn lẻ sang MBFF kết quả.

### 10.3 Tại sao MBFF tiết kiệm area và power

**Chia sẻ transistor**: Trong một MBFF, phần **clock network nội bộ** (các inverter và buffer tạo CLK, CLK_bar) được **chia sẻ** giữa tất cả các bit. Thay vì 8 bộ clock inverter riêng biệt, 1 bộ duy nhất phục vụ tất cả 8 bit. Điều này giảm trực tiếp số lượng transistor.

**Layout optimization**: Ở cấp độ transistor và cell, MBFF có layout được tối ưu hóa đặc biệt với các transistor được chia sẻ ở những điểm có thể, dẫn đến **total area nhỏ hơn** so với tổng area của các FF đơn lẻ.

**Ví dụ định lượng**: Một 2-bit MBFF có tổng area bằng khoảng **1.6× single FF** thay vì 2× → tiết kiệm ~20% area cho mỗi cặp FF được bank.

### 10.4 Lợi ích cho Clock Tree

**Giảm số lượng clock sink**: Clock Tree Synthesis (CTS) phải deliver clock đến tất cả clock pins. 8000 Single-bit FF → 8000 clock sinks. Nếu bank thành 2000 × 4-bit MBFF → chỉ còn 2000 clock sinks. CTS phải drive ít fanout hơn → ít buffer hơn → **clock tree wire length ngắn hơn → clock power giảm**.

**Cải thiện clock Skew**: Ít buffer → cây clock đơn giản hơn → dễ balance hơn → **Skew nhỏ hơn**. Skew nhỏ hơn cho phép tighter Setup/Hold margins → có thể tăng frequency hoặc có thêm timing margin cho data path.

### 10.5 Timing improvement từ MBFF

Skew nhỏ hơn trực tiếp cải thiện **Setup Slack**:

$$\text{Setup Slack} = T_{\text{clock}} - T_{\text{data}} - T_{\text{setup}} + \text{Skew}$$

Khi Skew giảm, ít credit timing từ Skew hơn (nếu borrow Skew) hoặc ít bị penalize hơn → tổng Slack cải thiện.

### 10.6 Thực hành: Kiểm tra MBFF opportunity

Lệnh `report_multibit` cho thấy thống kê hiện tại của design: số Single-bit FF, số Multi-bit FF hiện có, số bit chưa được merge. Nếu "Multibit Conversion %" = 0.00% (như ví dụ trong slide), có nghĩa là library không có MBFF Cell, hoặc không có FF nào đủ điều kiện bank.

---

## 11. Congestion — Phân Tích Và Giải Quyết

### 11.1 Định nghĩa chính xác của Congestion

**Congestion** (hay "routing congestion") xảy ra khi **số routing track cần thiết để kết nối tất cả Net trong một khu vực nhất định lớn hơn số routing track có sẵn** tại khu vực đó.

Định nghĩa định lượng: $$\text{Horizontal Congestion} = \text{Available horizontal tracks} - \text{Required horizontal tracks}$$ $$\text{Vertical Congestion} = \text{Available vertical tracks} - \text{Required vertical tracks}$$

Khi giá trị này âm → **overflow** → khu vực đó congested.

**GCELL (Global Cell)**: Công cụ routing chia die thành lưới GCELL — mỗi GCELL là một vùng hình chữ nhật nhỏ. Congestion được đo và báo cáo theo từng GCELL. Hình ảnh congestion map dùng màu sắc để thể hiện mức độ congestion tại từng GCELL:

- Xanh đậm/xanh lá: ít congested
- Vàng, cam: bắt đầu có vấn đề
- Đỏ: rất congested, khó route

**Overflow**: Khi routing demand trong một GCELL vượt quá capacity, số track bị "tràn" gọi là overflow. Một GCELL có overflow = 3 nghĩa là cần thêm 3 track nữa mà không có. Routing tool buộc phải reroute các Net này sang đường vòng dài hơn → tăng wirelength → tăng delay và power.

### 11.2 Tại sao Congestion phải được giải quyết ở Placement stage

Nguyên tắc: **Chi phí sửa congestion tăng theo hàm mũ theo thời điểm trong design flow**.

- Tại Placement: Chi phí thấp — có thể adjust placement, add blockages, change strategy rồi re-run. Không mất công routing.
- Tại Routing: Rất tốn kém — phải unroute một phần design, di chuyển Cell (nếu có thể), re-route. Có thể gây ra DRC violations mới.
- Sau Sign-off: Cực kỳ tốn kém — có thể phải restart từ Floorplan hoặc thậm chí Synthesis.

### 11.3 Global Routing (Early Routing Estimation)

Trong Placement stage, công cụ thực hiện một bước gọi là **Early Global Routing** hoặc **Route Early Global** — đây **không phải là routing thực sự** mà là một bước ước lượng (estimation step):

- **Mục đích**: Ước lượng routing feasibility để hướng dẫn Placement optimization.
- **Granularity**: Sử dụng GCELL grid (coarse grid), không route từng dây riêng lẻ.
- **Output**: Ước lượng wirelength, xác định congestion hotspot, phân bổ horizontal/vertical routing resource cho từng GCELL.
- **Ảnh hưởng**: Kết quả early global routing phản hồi lại cho placement engine (GigaPlace trong Innovus) để điều chỉnh vị trí Cell nhằm giảm congestion.

Lệnh `route_early_global` được chạy sau `place_opt_design` để evaluate routability trước khi quyết định có cần thêm bước tối ưu không.

### 11.4 Đánh giá Congestion: hai metrics

**Overflow Score** (từ `report_congestion -overflow`):

- Đo tỷ lệ toàn cục: trên tổng số GCELL trong die, có bao nhiêu % GCELL bị overflow.
- Thang đo: 0% → 100% (hoặc báo theo H và V riêng biệt).
- Ngưỡng chấp nhận được: **< 1% overflow** → design thường có thể route được.
- Ngưỡng cảnh báo: 1-5% overflow → sẽ có routing difficulties, DRC có thể khó clean.
- Ngưỡng nguy hiểm: > 5% overflow → design rất khó route, gần như chắc chắn sẽ có DRC violations sau routing.

**Hotspot Score** (từ `report_congestion -hotspot`):

- Đo mức độ nghiêm trọng của vùng tệ nhất, không phải trung bình.
- Tool báo cáo top-N khu vực congested nhất (bounding box + hotspot score).
- Normalized hotspot score: tổng hợp của routing demand excess trong khu vực đó.
- Ngưỡng chấp nhận được: **< 100** cho hotspot score normalized.
- Ngưỡng không thể route: **> 100** → khu vực đó fundamentally không thể route với Placement hiện tại.

Ví dụ từ slide: Hotspot top-1 = 117.25 → vượt ngưỡng 100 → khu vực đó cần được sửa. Overflow = 0.72% H + 2.76% V → tổng overflow ≈ 1.74% → ở mức cần chú ý nhưng chưa quá tệ.

### 11.5 Nguyên nhân của Congestion

**1. High Standard Cell density trong một khu vực nhỏ**: Quá nhiều Cell được đặt gần nhau → wires phải chen vào không gian hẹp.

**2. High standard cell pin density**: Một số Cell có nhiều Pin (ví dụ complex gate, wide MBFF) → pin density cao → routing rất khó vì phải access từng Pin riêng lẻ.

**3. Standard Cells gần Macro boundaries**: Tại cạnh của Macro (RAM, ROM, IP block), tất cả Port của Macro phải được kết nối → routing density tại boundary rất cao. Cells đặt sát cạnh Macro làm tình huống tệ hơn.

**4. High pin density tại cạnh Macro**: Tương tự điểm 3 — Macro có nhiều Port tập trung ở một cạnh → cạnh đó cực kỳ congested.

**5. Poor Floorplan**: Nếu các Macro được đặt không hợp lý (ví dụ: hai Macro liên quan đặt ở hai góc đối diện của die), các Net kết nối chúng phải đi qua toàn bộ die → tạo ra routing congestion trên toàn bộ đường đi đó.

**6. Poor Synthesis**: Synthesis không có physical awareness → tạo ra logic cấu trúc với high fan-in/fan-out không tự nhiên → nhiều Net crossing → congestion.

**7. Poor PG Grid strategy**: Quá nhiều VDD/VSS stripes trên metal layers được dùng cho signal routing → giảm available routing resources → effective congestion tăng.

### 11.6 Kỹ thuật giảm Congestion — Placement Blockages

**Hard Blockage**:

- **Cơ chế**: Không cho phép bất kỳ Standard Cell nào được đặt trong vùng được xác định.
- **Mục đích**: Tạo "khoảng trống vật lý" buộc routing tool có không gian để đi dây.
- **Tradeoff**: Giảm effective placement area → utilization tổng thể tăng lên (nếu die size cố định) → có thể gây ra congestion ở nơi khác nếu không cân bằng.
- **Use case**: Vùng quanh analog block cần isolation; vùng có nhiều PG stripes; vùng có IR-drop cao cần giảm current demand.

**Partial Blockage (Soft Blockage với density limit)**:

- **Cơ chế**: Cho phép Placement trong vùng nhưng giới hạn mật độ (density) không vượt quá một ngưỡng (ví dụ: 50%).
- **Mục đích**: Giảm density nhẹ nhàng hơn, không loại bỏ hoàn toàn → ít ảnh hưởng đến timing hơn Hard Blockage.
- **Use case**: Vùng có congestion vừa phải; vùng transition giữa khu dense và khu trống.

**Soft Blockage**:

- **Cơ chế**: Chỉ cho phép một loại Cell nhất định (ví dụ: Buffer, Inverter, clock gater) đặt trong vùng này; block tất cả Cell khác.
- **Mục đích**: Dành không gian cho các Cell cần thiết (ví dụ: buffer cho timing fix sau CTS) trong khi ngăn Cell logic thông thường chiếm chỗ.
- **Use case**: Vùng dành cho clock buffer insertion sau CTS.

**Macro-only Blockage**: Ngăn Macro (RAM, IP block) được đặt vào vùng cụ thể, nhưng vẫn cho phép Standard Cell.

### 11.7 Kỹ thuật giảm Congestion — Halo (Keep-out Region)

**Halo** là một vùng hình chữ nhật bao quanh Macro/memory, trong đó không cho phép Standard Cell nào được đặt:

```
+------------------------------------------+
|  Halo (no Standard Cell allowed here)    |
|   +---------------------------+           |
|   |                           |           |
|   |     Macro / Memory        |           |
|   |                           |           |
|   +---------------------------+           |
|                                           |
+------------------------------------------+
```

**Tại sao cần Halo**:

- Ports của Macro thường tập trung ở các cạnh. Khu vực gần cạnh cần không gian để route các connection từ Port.
- Standard Cell đặt sát cạnh Macro sẽ "cắt đứt" không gian routing, khiến router không thể tiếp cận các Port.
- Halo tạo "buffer zone" cho routing.

**Thông số Halo**: Kích thước Halo theo bốn hướng `{left bottom right top}` tính bằng micron. Ví dụ `{17 17 17 17}` = 17μm mỗi bên. Giá trị hợp lý phụ thuộc vào mật độ Port của Macro và số routing layer có sẵn.

### 11.8 Kỹ thuật giảm Congestion — Cell Padding

**Cell Padding** là thêm spacing constraint xung quanh một loại Cell cụ thể để ngăn các Cell khác đặt quá gần:

```
+--------------------------------+
| Padding (forced empty space)   |
|   +------------------------+   |
|   |  Cell (e.g., SDFFQX2)  |   |
|   +------------------------+   |
|                                |
+--------------------------------+
```

**Mục đích và use case**:

- Flip-flop phức tạp (SDFQ = Scan D Flip-Flop) có rất nhiều Pin (D, Q, CK, SE, SI) → cần routing access từ nhiều hướng → spacing giúp đảm bảo có đủ track cho routing.
- MBFF (Multi-Bit Flip-Flop) có đặc biệt nhiều Pin → cần padding nhiều hơn.
- Complex gate (AOI, OAI) với nhiều input.

**Thông số**: Padding riêng biệt cho từng hướng: left, right, top, bottom — tính bằng đơn vị site width (thường = 0.19μm ở 45nm node).

**Cơ chế hoạt động**: Legalization engine coi phần padding là phần của Cell khi kiểm tra overlap. Kết quả là hai Cell SDFFQX2 sẽ bị cách nhau tối thiểu `left_padding + right_padding` units.

### 11.9 Kỹ thuật giảm Congestion — Congestion-Driven Placement

**Timing-driven vs Congestion-driven**: Sự khác biệt căn bản:

**Timing-driven Placement** (mặc định):

- Objective: Minimize wirelength (đặc biệt trên critical path) để minimize delay.
- Kết quả: Cells kết nối với nhau được đặt gần nhau → wirelength ngắn → timing tốt.
- Side effect: Các Cell "cluster" lại thành vùng mật độ cao → congestion tăng.
- Analogy: Dân số tập trung ở thành phố → đường phố kẹt xe.

**Congestion-driven Placement**:

- Objective: Phân bố Cell **đều đặn** trên toàn diện tích có sẵn, tránh cluster.
- Kết quả: Mật độ đồng đều → không còn hotspot congestion.
- Side effect: Một số Net có wirelength dài hơn → timing xấu hơn.
- Analogy: Dân số phân tán về nông thôn → ít kẹt xe hơn nhưng di chuyển xa hơn.

**Khi nào dùng Congestion-driven**: Chỉ khi design **có đủ Timing Slack** (positive WNS/TNS lớn) để "trả" cho sự tăng delay do wirelength dài hơn. Nếu design đã gần timed (Slack ≈ 0), chuyển sang congestion-driven có thể tạo ra Timing violations mới.

**Thông số key**:

- `place_global_cong_effort high`: Tăng nỗ lực phân tích và giảm congestion trong global placement.
- `place_global_uniform_density true`: Buộc placer phải distribute Cell đều đặn trên die.

### 11.10 Kỹ thuật giảm Congestion — Power Grid Adjustment

**Vấn đề**: Mỗi Power/Ground stripe (PG stripe) trên metal layer chiếm một số routing track. Nếu có quá nhiều PG stripes dày đặc trên một layer được dùng chủ yếu cho signal routing, effective routing resource cho signals bị giảm đáng kể.

**Giải pháp**: Giảm số lượng PG stripes cục bộ trong khu vực congested, hoặc merge nhiều stripe mỏng thành ít stripe dày hơn (cùng total cross-section area để đảm bảo EM requirement).

**Ràng buộc quan trọng**: Không thể giảm PG grid tùy tiện. PG grid phải **đáp ứng EM (Electromigration) và IR-drop requirements**:

- EM: Current density trong mỗi metal wire không vượt quá giới hạn của process → cần đủ metal cross-section.
- IR-drop: Voltage drop từ power supply đến Cell không vượt quá ngưỡng (thường 5-10% VDD) → cần đủ số lượng và kích thước stripes.

**Công cụ**: `analyze_rail` và `report_power_grid` để đánh giá EMIR compliance. **IR-Drop-Aware Placement**: Placer có thể tích hợp IR-drop information để tránh đặt quá nhiều high-current cell vào vùng voltage drop cao.

### 11.11 Kỹ thuật giảm Congestion — Adjusting Floorplan

Khi congestion analysis cho thấy hotspot tập trung ở vùng cụ thể do **cách đặt Macro** (ví dụ: Memory đặt ở giữa die → Standard Cell bị dồn vào 4 góc → congestion ở boundary), giải pháp dứt điểm là **quay lại Floorplan stage** và điều chỉnh vị trí Macro.

Đây là bước **tốn kém nhất** trong các giải pháp congestion vì phải restart toàn bộ Placement flow. Tuy nhiên đôi khi không có lựa chọn nào khác khi root cause là Floorplan.

**Nguyên tắc tốt từ Floorplan** (liên kết với L7):

- Macro liên quan về mặt logic (ví dụ CPU core và L1 Cache) nên đặt gần nhau.
- Các Port ở cạnh Macro nên "nhìn về" vùng mà cells kết nối với chúng sẽ được đặt.
- Không để hai lớn Macro "kẹp" Standard Cell area ở giữa.

### 11.12 Kỹ thuật giảm Congestion — Physically-Aware Synthesis

**Root cause từ Synthesis**: Synthesis truyền thống không có thông tin về physical layout — nó không biết cell A và cell B sẽ cách nhau bao xa trên die thực tế. Kết quả:

- Logic có thể được structured theo cách tạo ra nhiều cross-chip connections.
- High-fanout net không được buffered đúng cách vì không biết wirelength thực tế.
- Cell placement sau đó dễ congested vì bản chất logic đã "xấu về mặt vật lý."

**Physically-Aware Synthesis** giải quyết bằng cách **cung cấp Floorplan/DEF information cho Synthesis tool** (ví dụ: Cadence Genus):

```
Traditional Flow: RTL → Synthesis → Netlist → PD (Floorplan → Placement → ...)

Physically-Aware Flow: 
RTL → Synthesis (với DEF từ Floorplan) → Netlist → PD
                    ↑                              ↓
                    ←←←← DEF với Placement data ←←←←
```

Bằng cách đọc `floorplan.def` vào Synthesis, tool biết vị trí tương đối của các Macro, kích thước Core area, và có thể ước lượng wirelength thực tế → tạo ra Netlist có tính physical-awareness:

- Tránh tạo long net kết nối hai điểm xa nhau nếu có cách implement logic khác.
- Buffer high-fanout net ngay từ Synthesis với drive strength phù hợp.
- Structure logic để minimize cross-die connections.

---

## 12. Outputs của Placement Stage và Tiêu Chí Đánh Giá

### 12.1 Các file output

Sau khi hoàn thành toàn bộ Post-Placement Optimization, design được export thành:

- **Optimized Netlist** (`.v`): Netlist Verilog đã được cập nhật với tất cả thay đổi từ optimization (cell swap, buffer insertion/removal, MBFF banking, Tie Cell, Spare Cell insertion).
- **Optimized DEF** (`.def`): Layout database với vị trí chính xác của tất cả Cell.
- **Database** (`.db`): Cadence Innovus database chứa toàn bộ design state.

### 12.2 Tiêu chí đánh giá chất lượng Placement — Exit Criteria

Một Placement được coi là **ready for CTS** khi thỏa mãn đồng thời tất cả điều kiện sau:

**Tiêu chí 1: Tất cả Cell đã được legalize**

- Không có Cell overlap (hai Cell cùng chiếm chỗ trên die).
- Không có unplaced Cell (Cell chưa được đặt vào die).
- Tất cả Cell phải align với site grid.
- Kiểm tra: `check_place` phải pass với 0 violations.

**Tiêu chí 2: Timing đạt yêu cầu (No hoặc minor timing violations)**

- **WNS (Worst Negative Slack)**: Tốt nhất ≥ 0. Ở giai đoạn Pre-CTS, timing được phân tích với ideal clock (không có clock tree delay) → sau CTS, timing sẽ xấu hơn do clock tree insertion delay. Do đó một số engineer yêu cầu WNS Pre-CTS phải dương với margin nhất định.
- **TNS (Total Negative Slack)**: Tổng của tất cả negative slack trên tất cả violating paths → phải = 0 hoặc rất nhỏ.
- Kiểm tra: `time_design -pre_cts`, `report_timing -summary`, `report_qor`.

**Tiêu chí 3: Không có DRV violations (trừ clock nets)** Ba loại DRV cần clean:

- **Max fanout**: Số output load connections của một Net không vượt quá giới hạn. Quá nhiều fanout → drive strength không đủ → transition time tăng (slew degradation) → có thể vi phạm Timing ở downstream.
- **Max capacitance**: Tổng capacitance load của một Net không vượt quá giới hạn. Tương tự max fanout về ảnh hưởng.
- **Max transition (Max slew)**: Transition time tại input/output của Cell không vượt quá giới hạn. Transition quá chậm → short-circuit power tăng, internal delay tăng, có thể vi phạm Timing.

**Tại sao exempt clock nets**: Clock nets tại Pre-CTS stage có thể có DRV violations vì clock tree chưa được built. Sau CTS, CTS tool sẽ insert clock buffers để fix clock DRV.

**Tiêu chí 4: Routability (congestion trong ngưỡng)**

- Global Overflow < 1%.
- Hotspot score < 100 cho tất cả top hotspots.

**Tiêu chí 5: Không có EMIR violations**

- IR-drop: Voltage drop từ supply đến Cell không vượt quá ngưỡng (thường 5% VDD).
- Electromigration: Current density trong PG wire không vượt quá giới hạn process.
- Kiểm tra: `analyze_rail`, `verify_pg_nets`.

**Tiêu chí 6: PPA requirements met**

- Power nằm trong budget đã định.
- Area không vượt quá target.
- Performance (timing) đạt spec.

### 12.3 Clock analysis tại Placement stage

Mặc dù CTS chưa chạy, cần kiểm tra một số thông tin clock để chuẩn bị:

- Số lượng clock domain và loại clock (propagated, generated).
- Clock gating efficiency: Tỷ lệ Sequential Cell được gated bởi ICG so với total Sequential Cell. Efficiency cao (>70%) là tốt.
- Clock power estimate: Ước lượng clock power với ideal clock → baseline trước CTS.
- Ràng buộc clock: Không được có max_transition violation trên clock nets (hoặc nếu có, CTS phải fix được).

---

## 13. Tổng hợp: Placement Flow Hoàn Chỉnh Và Vị Trí Trong Design Flow Tổng Thể

```
Data Import
    ↓
Floor Planning  (xác định die size, Macro placement, PG grid)
    ↓
Placement ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←← ITERATIONS
    ↓                                                         ↑
    ├── Set Up Placement Strategies          Power Optimization|
    ├── HFNS, Scan Reordering                Area Optimization|
    ├── DRV Fixing                            MBFF Optimization|
    ├── Timing Optimization (Pre-CTS)     Tie/Spare Cell Ins. |
    ├── Power/Area Optimization          Congestion Optimization→→
    ├── Tie Cell Insertion                                     
    ├── Spare Cell Insertion               
    ├── MBFF Optimization
    └── Congestion Analysis
              ↓
[Exit Criteria Check]
All Cells Legally Placed ✓
Routable (minimal congestion) ✓
No DRV Violations ✓
No EMIR Violations ✓
PPA Requirements Met ✓
              ↓
CTS (Clock Tree Synthesis)
    ↓
Routing
    ↓
Data Export (Tape-out)
```

### Mối liên hệ giữa các khái niệm trong L7 và L8

|Khái niệm từ L7|Kết nối với L8|
|---|---|
|Global Placement (minimize HPWL)|Wirelength ngắn → C thấp → $P_{\text{switching}}$ thấp|
|HFNS (High Fanout Net Synthesis)|High fanout → max_fanout DRV → cần buffer → tăng Power|
|DRV fixing (Pre-CTS)|DRV clean là exit criteria của Placement|
|Floorplan (Macro placement)|Congestion root cause — poor Floorplan → restart|
|set_dont_touch icg_reg_*|ICG Cell không bị move → clock gating integrity|
|place_global_max_density 0.7|70% utilization → 30% whitespace → routing space|

---

## 14. Công cụ EDA (Cadence Innovus) — Kiến trúc Engine

Cadence Innovus sử dụng các engine chuyên dụng:

- **GigaPlace**: Placement engine — thực hiện Global và Detail Placement.
- **GigaOpt**: Optimization engine — thực hiện Timing/DRV/Area optimization (`opt_design`).
- **PowerOpt**: Power optimization engine — thực hiện leakage/dynamic power optimization (`opt_power`).
- **NanoRoute**: Routing engine — thực hiện Global và Detail Routing sau CTS.
- **CCOpt**: Clock-tree engine — thực hiện CTS với concurrent clock/data optimization.

Mỗi lệnh optimization (`opt_design -pre_cts`, `opt_power -pre_cts`) gọi vào các engine tương ứng với thông tin timing analysis đã được set up (OCV, CPPR, process node).

---

## 15. Mối Liên Kết Khái Niệm — Sơ Đồ Tư Duy

```
POWER
├── Leakage (Static)
│   ├── Subthreshold leakage → controlled by VT selection (HVT/SVT/LVT)
│   ├── Gate oxide leakage → worse at advanced nodes, mitigated by high-k
│   └── Junction leakage → minor, less node-dependent
│
├── Dynamic
│   ├── Switching: α·C·V²·f
│   │   ├── α → Clock Gating (ICG) reduces effective α
│   │   ├── C → Wirelength optimization reduces wire cap
│   │   └── f, V → Architecture/system level (not PD)
│   └── Internal (Short-circuit) → Slew control via DRV fixing
│
└── Optimization Techniques
    ├── VT Swapping (LVT→HVT on non-critical path)
    ├── Cell Downsizing (reduce leakage + cap)
    ├── Buffer removal (reduce switching nodes)
    ├── ICG insertion (reduce clock switching)
    ├── MBFF banking (share clock buffer → reduce clock power)
    └── Wirelength reduction (reduce wire cap → reduce dynamic)

AREA
├── Optimization Techniques
│   ├── Cell Downsizing
│   └── Buffer/Inverter removal
└── Impact: less leakage (less transistor area), less dynamic (less cap)

CONGESTION
├── Causes
│   ├── High cell density → blockages, partial blockage
│   ├── Macro boundary → Halo
│   ├── High pin density cells → Cell Padding
│   ├── Poor Floorplan → Floorplan update
│   ├── Poor Synthesis → Physically-aware synthesis
│   └── Dense PG grid → Power grid adjustment
└── Strategy Switch
    └── Timing-driven → Congestion-driven (when Slack allows)

RELIABILITY
├── Tie Cells → protect gate oxide (antenna + surge)
└── Spare Cells → enable post-tapeout ECO without re-spin

SIGN-OFF READINESS
├── Timing: WNS≥0 (Pre-CTS), TNS=0
├── DRV: max_cap, max_tran, max_fanout all clean (except clock)
├── Congestion: Overflow<1%, Hotspot<100
├── EMIR: IR-drop OK, EM OK
└── Legality: No overlap, no unplaced cell
```