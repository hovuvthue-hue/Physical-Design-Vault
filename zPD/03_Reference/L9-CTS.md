# Clock Tree Synthesis (CTS) — Part I

## 1. Tổng quan về CTS

### 1.1 Định nghĩa

CTS (Clock Tree Synthesis) là quá trình insert các clock buffer và/hoặc inverter vào giữa clock source và clock pin của các sequential cell, nhằm mục tiêu tối thiểu hóa clock skew và insertion delay. Trước khi thực hiện CTS, tất cả clock pin của các sequential cell được drive trực tiếp bởi một clock source duy nhất — điều này không thực tế vì một driver đơn không thể đáp ứng đủ fanout và đảm bảo clock arrival time đồng đều trên toàn chip.

Điểm bắt đầu của CTS là clock source (gốc của clock tree), và các điểm kết thúc (endpoint) là clock pin của các sequential cell (Flip-flop, Latch, Macro).

### 1.2 Tầm quan trọng

CTS là một trong những giai đoạn quan trọng nhất trong Physical Design flow, nằm giữa Placement và Routing. CTS QoR (Quality of Results) ảnh hưởng trực tiếp và mạnh mẽ đến timing convergence và power consumption của toàn chip.

### 1.3 CTS Flow tổng quát

CTS là một quá trình lặp (iterative), gồm các bước chính:

1. **Sanity checks** sau Placement: kiểm tra tính hợp lệ của design trước khi bắt đầu CTS.
2. Nếu sanity check sạch → tiến hành CTS với ba hoạt động cốt lõi:
    - **Clustering-Balancing**: nhóm các sink pin và cân bằng clock path.
    - **Route clock nets**: routing các net thuộc clock tree.
    - **Optimization**: tối ưu hóa skew, insertion delay, transition.
3. Kiểm tra xem kết quả có đạt yêu cầu (Meet Requirements) không. Nếu chưa → lặp lại bước CTS.
4. Nếu đạt → chuyển sang Routing.

---

## 2. Mục tiêu của CTS

Các mục tiêu được sắp xếp theo thứ tự ưu tiên:

### 2.1 Đạt Clock Tree Targets (ưu tiên cao nhất)

- **Cân bằng clock skew và insertion delay** giữa các sink.
- **Tối thiểu hóa clock skew** để các Flip-flop nhận clock gần như đồng thời.
- **Tối thiểu hóa insertion delay** để giảm overhead trên timing path.

### 2.2 Đáp ứng Clock Tree Design Rules

- Tuân thủ giới hạn Max Transition, Max Fanout, Max Capacitance trên clock net.
- Đảm bảo Minimum Pulse Width của clock signal.

### 2.3 Đáp ứng Timing Constraints

- Setup timing và Hold timing phải được thỏa mãn sau CTS.

### 2.4 Tối thiểu hóa Power Consumption

- Giảm cả Dynamic power (do switching activity) và Leakage power.

### 2.5 Tối thiểu hóa Noise trên Clock Net

- Giảm crosstalk và clock glitch — những hiện tượng gây suy giảm signal integrity và tạo ra timing error.

---

## 3. Inputs của CTS

CTS nhận vào một tập hợp dữ liệu gọi chung là **Placement Optimized DB**, bao gồm:

- **Placement Netlist**: mô tả kết nối logic sau Placement.
- **Physical Libraries (LEF)**: định nghĩa thông tin vật lý của cell (kích thước, pin location, layer rules).
- **Placement DEF**: chứa vị trí đã được placed của tất cả cell.
- **Multi-Mode Multi-Corner (MMMC)**:
    - Timing libraries (`.lib`): chứa thông tin delay, setup/hold time của cell theo từng PVT corner.
    - Timing constraints (SDC): định nghĩa clock, path exceptions, I/O constraints.
    - RC corners: mô hình điện trở-tụ điện của wire theo từng process/temperature corner.
- **Clock Tree Spec**: file spec định nghĩa toàn bộ yêu cầu và constraint cho CTS.

---

## 4. Pre-CTS Sanity Checks

Trước khi tiến hành CTS, cần xác nhận rằng design đã đáp ứng đủ các điều kiện tối thiểu:

- **Placement hợp lệ**: tất cả Standard Cell và Macro phải được placed hợp lệ (không overlap, không vi phạm design rule về vị trí).
- **Congestion trong giới hạn cho phép**: nếu congestion quá cao, CTS và Routing sẽ không thể hoàn thành.
- **Setup timing**: không có setup violation nghiêm trọng, hoặc chỉ còn minor violation chấp nhận được. Setup violation nặng trước CTS thường không thể khắc phục trong CTS.
- **DRV (Design Rule Violation) đã được clean** — ngoại trừ trên clock net (vì clock net chưa được built):
    - Max Fanout
    - Max Transition
    - Max Capacitance
- **Clock definitions phải được khai báo đầy đủ và chính xác** trong SDC, bao gồm cả primary clock (`create_clock`) và generated clock (`create_generated_clock`). Nếu clock không được định nghĩa đúng, CTS tool sẽ không biết phải build clock tree cho clock nào và đến đâu.

---

## 5. CTS Terminologies

### 5.1 Clock Insertion Delay (Clock Latency)

Clock insertion delay (còn gọi là clock latency) là thời gian cần thiết để clock signal lan truyền từ clock source (gốc của clock tree) đến clock input pin của sequential cell hoặc macro.

Clock insertion delay được chia thành hai thành phần:

**Source Insertion Delay**: thời gian clock lan truyền từ clock source (thường là PLL hoặc oscillator) đến clock definition point (thường là input port của block).

**Network Insertion Delay**: thời gian clock lan truyền từ clock definition point đến clock pin của từng Flip-flop bên trong design.

$$\text{Total Insertion Delay} = \text{Source Insertion Delay} + \text{Network Insertion Delay}$$

Ý nghĩa thực tiễn: insertion delay lớn sẽ tiêu thụ một phần timing budget của clock cycle, gián tiếp làm giảm slack của setup path.

---

### 5.2 Clock Skew

Clock skew là sự chênh lệch thời gian clock đến giữa hai sequential element (Flip-flop) bất kỳ trong design.

$$\text{Clock Skew} = T_{\text{capture\_arrival}} - T_{\text{launch\_arrival}}$$

**Nguyên nhân gây skew**:

- Chênh lệch độ dài clock path (unequal path delays).
- Số lượng buffer/inverter được insert không đồng đều.
- Sự khác biệt về wire length và RC khi routing.
- Chênh lệch tải (clock loading imbalance) giữa các nhánh.

**Phân loại theo tác động đến timing**:

_Positive Skew_: capture clock đến **sau** launch clock → $T_{\text{skew}} > 0$. Làm tăng timing window cho setup, nhưng nếu quá lớn sẽ gây hold violation.

_Negative Skew_: capture clock đến **trước** launch clock → $T_{\text{skew}} < 0$. Cải thiện hold timing, nhưng thu hẹp setup window và có thể gây setup violation.

**Phân loại theo phạm vi**:

**Global Clock Skew**: chênh lệch clock arrival time giữa hai Flip-flop không liên quan (non-related), hoặc giữa đường clock dài nhất và ngắn nhất trong toàn chip.

$$\text{Global Skew} = T_{\text{longest\_path}} - T_{\text{shortest\_path}}$$

Global skew ảnh hưởng đến timing convergence tổng thể và CTS QoR của toàn chip. Global skew lớn sẽ làm tăng setup/hold violation và clock uncertainty.

**Local Clock Skew**: chênh lệch clock arrival time giữa launch Flip-flop và capture Flip-flop được kết nối với nhau bởi một timing path cụ thể.

Local skew ảnh hưởng trực tiếp đến setup timing, hold timing, và quan hệ timing của datapath. CTS optimization tập trung chủ yếu vào việc giảm local skew có hại.

---

### 5.3 Setup Time và Hold Time

**Setup Time** ($T_{su}$): khoảng thời gian tối thiểu mà data input phải ổn định **trước** khi active clock edge đến tại Flip-flop. Đây là yêu cầu từ cell library, phụ thuộc vào technology và cell drive strength.

**Hold Time** ($T_{ho}$): khoảng thời gian tối thiểu mà data input phải ổn định **sau** khi active clock edge đến tại Flip-flop.

Hai thông số này là đặc tính vật lý của sequential cell và không thể thay đổi. Toàn bộ CTS và timing optimization đều nhằm đảm bảo data path thỏa mãn cả hai yêu cầu này.

---

### 5.4 Setup Timing Check

Setup timing check đảm bảo design có thể hoạt động đúng tại target clock frequency.

**Nguyên tắc phân tích**:

- Sử dụng **slow timing library** (worst-case cell delay).
- Sử dụng **worst RC corner** (maximum wire delay).
- Setup check được đánh giá tại **active clock edge thứ hai** (capture edge).
- Setup timing phụ thuộc vào clock frequency.

$$\text{Setup Slack} = (T_{\text{cycle}} - T_{\text{setup}}) - (T_{\text{clk→Q}} + T_{dp} + T_{\text{capture}} - T_{\text{launch}})$$

Trong đó:

- $T_{\text{cycle}}$: chu kỳ clock
- $T_{\text{setup}}$: setup time của capture Flip-flop
- $T_{\text{clk→Q}}$: clock-to-Q delay của launch Flip-flop
- $T_{dp}$: combinational logic delay trên datapath
- $T_{\text{capture}}$: clock arrival time tại capture Flip-flop
- $T_{\text{launch}}$: clock arrival time tại launch Flip-flop

Slack dương → timing met. Slack âm → violation.

**Mối liên hệ với skew**: positive skew ($T_{\text{capture}} > T_{\text{launch}}$) làm tăng setup slack vì nó mở rộng thời gian data có thể lan truyền. Negative skew làm giảm setup slack.

---

### 5.5 Hold Timing Check

Hold timing check đảm bảo Flip-flop capture và giữ data đúng sau active clock edge.

**Nguyên tắc phân tích**:

- Sử dụng **fast timing library** (best-case cell delay).
- Sử dụng **best RC corner** (minimum wire delay).
- Hold check được đánh giá tại **active clock edge đầu tiên** (cùng cycle với launch).
- Hold timing **không phụ thuộc** vào clock frequency — đây là điểm khác biệt quan trọng so với setup.

$$\text{Hold Slack} = T_{\text{clk→Q}} + T_{dp} - T_{ho} - (T_{\text{capture}} - T_{\text{launch}})$$

**Mối liên hệ với skew**: negative skew ($T_{\text{capture}} < T_{\text{launch}}$) giúp cải thiện hold slack. Positive skew làm giảm hold margin, và nếu quá lớn sẽ gây hold violation.

Vì hold violation không phụ thuộc frequency, nó phải được fix bằng cách insert thêm delay (buffer) vào datapath, không thể giải quyết bằng cách giảm frequency.

---

### 5.6 Positive và Negative Clock Skew — Chi tiết

**Positive Skew** ($T_{\text{skew}} > 0$: capture clock đến sau launch clock):

$$\text{Setup Slack} = T_{\text{clock}} - T_{\text{clk-q,FF1}} - T_{\text{comb}} - T_{\text{su,FF2}} + T_{\text{skew}}$$

$$\text{Hold Slack} = T_{\text{clk-q,FF1}} + T_{\text{comb}} - T_{\text{ho,FF2}} - T_{\text{skew}}$$

Positive skew cải thiện setup nhưng làm xấu hold.

**Negative Skew** ($T_{\text{skew}} < 0$: capture clock đến trước launch clock):

$$\text{Setup Slack} = T_{\text{clock}} - T_{\text{clk-q,FF1}} - T_{\text{comb}} - T_{\text{su,FF2}} - |T_{\text{skew}}|$$

$$\text{Hold Slack} = T_{\text{clk-q,FF1}} + T_{\text{comb}} - T_{\text{ho,FF2}} + |T_{\text{skew}}|$$

Negative skew thu hẹp setup window nhưng cải thiện hold margin.

---

### 5.7 Clock Jitter

Clock jitter là sự sai lệch của clock edge so với vị trí lý tưởng của nó theo thời gian. Jitter gây ra biến thiên trong clock period và clock arrival time theo từng chu kỳ.

Jitter quá lớn sẽ làm suy giảm setup timing, hold timing, clock stability, và timing convergence tổng thể.

**Nguồn gây jitter**:

- Crosstalk noise từ các net lân cận.
- Power supply variation (IR drop, voltage fluctuation).
- Thermal variation.
- PLL noise (phase noise của PLL nguồn).
- Process variation (độ không đồng đều trong manufacturing).
- Electromagnetic interference (EMI).

Jitter được mô hình hóa thông qua clock uncertainty trong STA và ảnh hưởng trực tiếp đến cả setup và hold slack.

---

### 5.8 Clock Uncertainty

Clock uncertainty định nghĩa cửa sổ thời gian (timing window) mà trong đó clock edge có thể xuất hiện. Đây là một thông số dự phòng (margin) được áp dụng trong STA để mô hình hóa các hiệu ứng không xác định được chính xác trước khi có thực tế layout hoàn chỉnh.

Clock uncertainty mô hình hóa các nguồn bất định sau:

- Clock jitter.
- Biến thiên clock skew.
- PVT variation.
- Margin bổ sung cho setup/hold.

Uncertainty lớn hơn → timing margin nhỏ hơn:

- **Setup uncertainty** làm giảm setup slack.
- **Hold uncertainty** làm giảm hold margin và có thể tạo thêm hold violation.

---

### 5.9 Crosstalk Effect

Crosstalk là hiện tượng coupling điện không mong muốn giữa các wire nằm gần nhau, chủ yếu thông qua capacitive coupling.

Khi net aggressor chuyển trạng thái (switching), nó có thể gây nhiễu lên net victim lân cận:

- **Speed up victim**: aggressor và victim chuyển cùng chiều → victim nhanh hơn bình thường.
- **Slow down victim**: aggressor và victim chuyển ngược chiều → victim chậm hơn bình thường (crosstalk delay).
- **Glitch trên victim**: victim đang ở trạng thái tĩnh nhưng bị noise đẩy vượt ngưỡng logic → tạo glitch giả.

Crosstalk đặc biệt nguy hiểm trên clock net vì nó có thể gây thêm jitter, làm thay đổi insertion delay không đồng đều giữa các branch, và tạo glitch lan truyền đến Flip-flop.

---

### 5.10 Clock Slew (Transition)

Clock slew (hay transition time) là thời gian cần thiết để clock signal chuyển trạng thái:

- **Rising transition**: từ 10% → 90% của $V_{DD}$.
- **Falling transition**: từ 90% → 10% của $V_{DD}$.

Slew quá lớn (slow transition) sẽ:

- Tăng propagation delay.
- Làm xấu clock skew.
- Tăng power consumption (thời gian short-circuit current dài hơn).
- Suy giảm signal integrity.

CTS cố gắng tối thiểu hóa và cân bằng clock slew trên toàn clock tree. Clock buffer được thiết kế đặc biệt để duy trì slew trong giới hạn cho phép.

---

### 5.11 Clock Duty Cycle

Duty cycle là tỉ lệ phần trăm của một clock period mà clock signal ở trạng thái active (high):

$$\text{Duty Cycle} = \frac{T_{\text{on}}}{T_{\text{period}}} \times 100\%$$

Lý tưởng là 50%, nghĩa là $T_{\text{on}} = T_{\text{off}}$.

**Tại sao duty cycle quan trọng**: Duty cycle degradation ảnh hưởng trực tiếp đến:

- Half-cycle timing path: các path launch ở rising edge và capture ở falling edge (hoặc ngược lại) chỉ có $T_{\text{period}}/2$ để propagate data.
- Minimum pulse width check.
- Latch-based design (latch transparent trong nửa cycle).
- Clock reliability nói chung.

**Nguyên nhân duty cycle degradation**:

- Mismatch giữa rise/fall transition trên clock path.
- Chênh lệch drive strength PMOS/NMOS.
- OCV (On-Chip Variation): sự không đồng đều về process trong cùng một chip.
- Crosstalk và noise.
- Excessive clock buffering qua nhiều tầng.
- PVT variation.

---

### 5.12 Minimum Pulse Width

Tất cả sequential element đều yêu cầu clock pulse width tối thiểu để hoạt động đúng. Clock pulse width phải luôn lớn hơn ngưỡng tối thiểu được định nghĩa trong library của cell.

Yêu cầu minimum pulse width phụ thuộc vào:

- Technology node.
- Standard cell library.
- Operating voltage.
- Process conditions.

Vi phạm minimum pulse width có thể gây:

- Data capture sai.
- Metastability.
- Functional failure.

**Nguyên nhân gây vi phạm minimum pulse width**:

- Duty cycle degradation (đã phân tích trên).
- Clock slew quá lớn làm méo pulse.
- Skew imbalance giữa các branch.
- Clock signal distortion do crosstalk.

**Phương pháp khắc phục**: sử dụng balanced clock buffer và matched inverter structure (pair số chẵn inverter) trong CTS để hạn chế pulse distortion. Insert thêm buffer hoặc inverter tại vị trí cụ thể trên clock tree để điều chỉnh pulse width.

---

### 5.13 Single-Cycle Path vs Half-Cycle Path

**Single-Cycle Path**: data được launch và capture bởi hai Flip-flop dùng cùng loại clock edge (positive-to-positive hoặc negative-to-negative). Data có toàn bộ một clock cycle để propagate. Setup và hold check trong một full cycle.

**Half-Cycle Path**: data được launch và capture bởi hai Flip-flop dùng hai loại clock edge đối lập (positive-to-negative hoặc negative-to-positive). Data chỉ có $T_{\text{period}}/2$ để propagate.

Hệ quả:

- Setup timing bị **siết chặt** còn một nửa chu kỳ — đây là constraint rất khắc nghiệt với các path cao tần.
- Hold timing được **nới lỏng** một nửa chu kỳ — dễ đáp ứng hơn.

Half-cycle path thường xuất hiện trong design có mixed-edge register interface, ví dụ giữa positive-edge Flip-flop và negative-edge Latch.

---

### 5.14 Clock Skew Groups

Clock skew group định nghĩa một tập hợp clock sink pin phải được cân bằng với nhau trong quá trình CTS optimization. Tất cả sink pin trong cùng một skew group được tối ưu hóa cùng nhau để đạt insertion delay đồng đều, skew thấp, và synchronized clock arrival.

**Tại sao cần skew group**:

- Các clock domain khác nhau có thể yêu cầu cân bằng độc lập.
- Phân tách skew group giúp:
    - Cải thiện CTS convergence.
    - Giảm buffering không cần thiết (không phải balance những sink không có timing relationship với nhau).
    - Cải thiện timing closure.
    - Giảm power.
- Mỗi skew group có CTS constraint riêng (skew target, insertion delay target...).
- Các sink thuộc skew group khác nhau **không được cân bằng** với nhau.

Về mặt nguyên lý, nếu hai Flip-flop không có timing path kết nối với nhau (ở hai clock domain khác nhau hoặc cách xa nhau về logic), việc cân bằng skew giữa chúng không có ý nghĩa timing và chỉ tiêu tốn tài nguyên.

---

### 5.15 Clock Buffer vs Regular Buffer

**Clock Buffer**: được thiết kế đặc biệt với các đặc tính:

- **Balanced rise/fall transition**: thời gian rise và fall bằng nhau → duy trì duty cycle ~50%.
- **Symmetric delay**: delay như nhau ở cả hai chiều chuyển tiếp.
- **Low skew**: phù hợp cho clock distribution.
- **Stable duty cycle**: không làm suy giảm pulse width.

Clock buffer giúp tối thiểu hóa duty cycle degradation, pulse width distortion, clock jitter, và skew imbalance. Đây là lý do clock buffer **bắt buộc** được dùng trong CTS thay vì regular buffer.

**Regular Buffer**: buffer đa dụng cho datapath optimization.

- Không đảm bảo balanced rise/fall transition.
- Có thể gây asymmetric delay dẫn đến duty cycle distortion.
- **Không được dùng** để build clock tree trong CTS.

---

### 5.16 Clock Gating

Clock gating là kỹ thuật disable clock cho các register không hoạt động (idle/inactive) để giảm dynamic power consumption. Logic clock gating thường được implement bằng một Latch kết hợp cổng AND/NAND, hoặc một Integrated Clock Gating (ICG) cell chuyên dụng. ICG cell bao gồm một latch âm (negative-level sensitive latch), một cổng AND, và một test enable (TE) pin để hỗ trợ scan.

**Lợi ích**:

- Giảm dynamic power consumption đáng kể.
- Giảm switching activity trên clock net và downstream logic.
- Cải thiện power efficiency.

**Nhược điểm**:

- Tăng độ phức tạp của clock tree.
- Có thể làm tăng insertion delay.
- Có thể làm tăng clock skew.
- Thêm area overhead.
- Yêu cầu CTS optimization bổ sung để xử lý ICG cell đúng cách.

#### Clock Gating Check

Clock gating check là timing constraint đảm bảo rằng enable signal ổn định xung quanh active clock edge để tránh tạo clock glitch. Nếu enable signal thay đổi quá gần clock edge, glitch có thể xuất hiện trên gated clock output, gây trigger sai cho downstream Flip-flop.

Clock gating check có thể được:

- Tự động inference bởi EDA tool dựa trên cell topology.
- Hoặc được định nghĩa tường minh qua timing constraint.

Vi phạm clock gating check dẫn đến: clock glitch, trigger sai register, metastability, và functional failure.

---

### 5.17 Non-Default Rule (NDR)

NDR là custom routing rule override lên default routing constraint, định nghĩa các yêu cầu đặc biệt như:

- Wire width rộng hơn.
- Wire spacing lớn hơn.
- Shielding.
- Special via rule.

Khi NDR được apply cho một net, router bắt buộc phải tuân theo constraint đó cho net đó.

**Lý do dùng NDR cho clock net**:

_Cải thiện Signal Integrity_: spacing lớn hơn giảm crosstalk, coupling capacitance, và noise giữa clock net và các aggressor net lân cận.

_Cải thiện Reliability_: wire width rộng hơn giảm điện trở, từ đó giảm IR drop, electromigration (EM), và current density.

_Cải thiện Timing_: resistance thấp hơn → RC delay thấp hơn → insertion delay nhỏ hơn và đồng đều hơn → skew tốt hơn.

NDR thường được phân loại theo phần của clock tree: root nets (phần trên), internal nets (phần giữa), và sink nets (phần dưới). Mỗi phần có thể dùng NDR rule khác nhau phù hợp với yêu cầu và tải điện của từng phần.

---

## 6. CTS Exceptions

### 6.1 Tổng quan

CTS exceptions là các clock pin hoặc endpoint đặc biệt yêu cầu CTS tool xử lý theo cách khác với pin thông thường. Người thiết kế phải chỉ định tường minh cách xử lý các pin này.

### 6.2 Sink Pins

Sink pin là clock endpoint mà CTS sử dụng cho: clock balancing, skew optimization, và insertion delay control. Sink pin thường là: sequential cell clock pin, macro clock pin, hoặc clock gating sink pin. CTS tool tự động nhận diện các sink pin chuẩn.

### 6.3 Stop Pin

Stop pin là điểm cuối của clock tree nơi CTS dừng propagation và thực hiện delay balancing. CTS coi stop pin là valid clock sink endpoint cho skew balancing, insertion delay optimization, và clock latency control.

**Implicit Stop Pin**: được tự động nhận diện bởi CTS tool, gồm clock pin của Flip-flop, Latch, và Macro clock pin. CTS tự dừng tại các điểm này.

**Explicit Stop Pin**: do người thiết kế định nghĩa tường minh, thường dùng cho hierarchical CTS, generated clock boundary, macro boundary, hoặc custom clock structure.

Ý nghĩa: stop pin định nghĩa phạm vi balancing của CTS. Định nghĩa stop pin đúng giúp cải thiện skew balancing, kiểm soát insertion delay, và tránh buffer hóa không cần thiết.

### 6.4 Nonstop Pin

Nonstop pin là pin mà CTS tiếp tục propagate clock qua dù về mặt logic nó có thể bị coi là stop pin. CTS không terminate clock tree tại nonstop pin, mà tiếp tục: buffering, skew balancing, và insertion delay optimization về phía sau pin đó.

Ứng dụng phổ biến: Generated Clock Driver Pin và Clock Gating Cell Pin. Ví dụ, clock input pin của ICG cell (ICG/CLK) thường được set là nonstop để CTS tiếp tục propagate qua ICG và cân bằng downstream Flip-flop.

Ý nghĩa: nếu CTS dừng tại clock input của ICG, thì các Flip-flop downstream của ICG sẽ không được balance, gây ra skew không kiểm soát.

### 6.5 Exclude Pin

Exclude pin là clock pin bị loại trừ khỏi quá trình skew balancing và insertion delay optimization. CTS có thể vẫn propagate clock network qua pin này, nhưng pin đó không được coi là balancing endpoint.

Ứng dụng: non-critical clock sink, debug logic, test structure, special clock path mà không cần tối ưu timing chính xác.

**Phân biệt Exclude và Ignore**: Exclude pin không được dùng cho skew optimization nhưng clock network vẫn có thể propagate qua nó. Ignore pin thì CTS hoàn toàn bỏ qua — không propagate, không balance, không optimize.

### 6.6 Float Pin

Float pin là clock sink pin có user-defined insertion delay target (thường dùng cho macro, memory, hard IP block). Float pin được dùng khi clock vào macro nhưng bên trong macro có thêm internal insertion delay mà CTS không thể nhìn thấy. Người thiết kế cần bù trừ cho internal delay đó.

Ví dụ tính toán: $$\text{Float Pin Delay} = T_{\text{target\_insertion}} - T_{\text{memory\_internal\_delay}}$$ $$= 2.0\text{ ns} - 0.2\text{ ns} = 1.8\text{ ns}$$

CTS sẽ balance clock đến float pin với delay target 1.8 ns, nhờ đó tổng insertion delay đến register bên trong memory sẽ là 1.8 + 0.2 = 2.0 ns, bằng với target.

### 6.7 Ignore Pin

Ignore pin là pin mà CTS hoàn toàn bỏ qua trong suốt quá trình clock tree construction và optimization: không propagate clock, không balance skew, không optimize insertion delay. Các pin này bị loại hoàn toàn khỏi clock topology.

Mặc định, CTS thường treat các loại pin sau là ignore pin:

- Non-clock pin của sequential cell.
- Output port.
- Source pin của clock tree nằm trong clock domain fanout khác.

### 6.8 Through Pin

Through pin là pin mà CTS propagate clock signal qua đó trong khi tiếp tục build clock tree. CTS coi through pin là điểm propagation trung gian (intermediate), không phải sink endpoint. Through pin cho phép CTS tiếp tục buffer, balance, và optimize về phía sau.

---

## 7. CTS Specification (CTS Spec)

### 7.1 Định nghĩa và mục đích

CTS Spec file là file định nghĩa toàn bộ yêu cầu và constraint được sử dụng trong quá trình CTS optimization. CTS tool dựa vào spec file để kiểm soát:

- Clock topology (kiến trúc clock tree).
- Skew target.
- Insertion delay target.
- Transition/slew limit.
- Routing rule.
- CTS exception handling.

### 7.2 Nội dung điển hình của CTS Spec

**Clock Definitions**: định nghĩa primary clock và generated clock, chỉ định clock source, clock period. Generated clock phải được khai báo tường minh với master clock, nguồn gốc (divider, mux, gating logic...) và relationship (divide_by, multiply_by...).

**Clock Insertion Delay Target**: giá trị latency mong muốn từ clock source đến sink pin. Đây là target để CTS cố gắng đạt được khi build tree.

**Clock Skew Target**: giá trị skew tối đa cho phép giữa các sink trong cùng skew group. Giá trị này phải thực tế — quá chặt sẽ dẫn đến quá nhiều buffering và tăng power/area.

**Clock Transition (Slew) Constraint**: giới hạn max transition trên clock net. Nếu slew vượt giới hạn này, CTS sẽ insert thêm buffer để drive mạnh hơn và giảm transition time.

**Clock Buffer và Inverter List**: danh sách cell có thể được dùng để build clock tree. Chỉ clock buffer và clock inverter mới được phép — không dùng regular buffer. Việc chọn cell size phù hợp ảnh hưởng đến trade-off giữa insertion delay, fanout capability, power, và slew.

**Clock Skew Groups**: định nghĩa nhóm các sink cần được balance với nhau, gồm thông tin nguồn clock (source) và các sink thuộc nhóm đó.

**NDR Rules**: quy định routing rule cho clock net (width, spacing, shielding). Thường phân loại theo root, internal, và sink net.

**Default Timing Corner**: corner được dùng để đánh giá CTS trong quá trình optimization. Thường là slow corner (worst-case) cho setup-focused optimization.

**CTS Exceptions**: danh sách các pin đặc biệt và loại exception (stop, nonstop, exclude, ignore, through, float).

### 7.3 CTS Spec Generation Flow

Quy trình tạo CTS Spec là iterative:

1. Load SDC/MMMC và CTS settings.
2. Tool tự động generate initial CTS Spec từ clock definitions trong SDC.
3. Người thiết kế review và manually thêm/chỉnh sửa: exception pin, NDR rule, skew group, buffer list, constraint target...
4. Chạy CTS với spec đó.
5. Phân tích kết quả.
6. Refine CTS Spec nếu cần và lặp lại.

---

## 8. Clock Definitions — Primary và Generated Clock

**Primary Clock**: clock được tạo trực tiếp từ input port, PLL output, hoặc oscillator source. Đây là gốc của clock tree.

**Generated Clock**: clock được dẫn xuất từ primary clock thông qua: divider (chia tần số), mux (chọn clock), gating logic, PLL, hoặc sequential logic. Generated clock **bắt buộc** phải được khai báo tường minh trong SDC để:

- CTS biết đây là một clock mới cần build sub-tree.
- STA phân tích đúng clock relationship giữa các domain.
- Timing path crossing giữa master clock domain và generated clock domain được phân tích với phase relationship chính xác.

Nếu generated clock không được định nghĩa, STA có thể hiểu sai clock relationship, dẫn đến kết quả timing không chính xác.

---

## 9. NDR Rules cho CTS Routing

NDR rules cho CTS được phân loại theo vị trí trong clock tree:

**Root Nets** (từ clock source đến nhánh đầu tiên): thường có NDR nghiêm ngặt nhất — width 2x, spacing 2x, và có shielding. Lý do: root net drive toàn bộ clock tree nên reliability và SI là ưu tiên hàng đầu. Current density ở root lớn nhất → cần width rộng nhất để đảm bảo EM.

**Internal Nets** (các nhánh trung gian): NDR mức trung bình. Không nhất thiết shielding nhưng vẫn cần width/spacing lớn hơn để kiểm soát skew và SI.

**Sink Nets** (từ buffer cuối cùng đến clock pin của Flip-flop): có thể dùng NDR nhẹ hơn vì wire ngắn hơn và current nhỏ hơn. Tuy nhiên vẫn cần đảm bảo slew và DRC.

**Shielded Routing**: bổ sung thêm lớp wire VDD hoặc VSS chạy song song với clock net, tạo tường chắn tĩnh điện ngăn crosstalk từ aggressor net khác. Shielding cải thiện: signal integrity, jitter control, và timing stability — đặc biệt quan trọng với các clock net ở upper metal layer (độ coupling cao do wire dài và spacing hẹp hơn ở lower layer bị chiếm bởi data net).

---

## 10. Types of Clock Trees

Kiến trúc clock tree được chọn dựa trên yêu cầu về skew, power, routing complexity, scalability, và performance target.

### 10.1 Conventional Clock Tree (Single Source)

Cấu trúc: sử dụng một clock root source duy nhất, buffer và inverter được insert theo dạng cây phân cấp (hierarchical tree) để drive tất cả sink register.

**Đặc điểm và ưu điểm**:

- Phù hợp với thiết kế tần số thấp đến trung bình.
- Power consumption thấp hơn so với clock mesh.
- Dễ implement clock gating.
- Cấu trúc CTS đơn giản, dễ debug và phân tích.

**Hạn chế**:

- Nhạy cảm với PVT variation vì toàn bộ tree phụ thuộc vào một path từ root.
- Insertion delay lớn với chip lớn do phải đi qua nhiều tầng buffer.
- Skew variation lớn khi clock path dài.
- Khả năng scale kém với chip rất lớn.

### 10.2 Clock Mesh

Cấu trúc: sử dụng mạng metal dày đặc hình lưới (grid) để phân phối clock trên toàn chip. Nhiều clock driver cấp đồng thời vào mesh, tạo nhiều đường propagation song song.

**Ưu điểm**:

- Skew rất thấp nhờ có nhiều parallel path — clock signal có thể "đi tắt" qua nhiều hướng.
- Khả năng chống OCV cao: vì tất cả Flip-flop đều nhìn vào cùng một mesh, biến thiên local process chỉ ảnh hưởng nhỏ đến arrival time.
- Timing stability và jitter tolerance tốt.

**Nhược điểm**:

- Dynamic power rất cao — toàn bộ mesh capacitance switch mỗi cycle.
- Tiêu tốn lớn routing resource và gây metal congestion.
- Clock gating khó implement vì cấu trúc lưới không có điểm cắt rõ ràng.
- Thời gian implement và optimize dài hơn.

### 10.3 H-Tree

Cấu trúc: kiến trúc clock distribution đối xứng theo hình chữ H. Clock network được chia đệ quy (recursively) thành các nhánh cân bằng với độ dài path bằng nhau.

**Ưu điểm**:

- Đạt skew thấp tự nhiên do routing đối xứng — tất cả sink có cùng độ dài clock path.
- OCV tolerance tốt hơn conventional tree nhờ nhiều common path hơn.
- Power thấp hơn clock mesh.
- Insertion delay có thể dự đoán trước.

**Nhược điểm**:

- Insertion delay cao hơn clock mesh (path dài hơn đến sink).
- Khó implement với floorplan bất đối xứng hoặc có nhiều macro blockage.
- Yêu cầu super clock buffer (drive strength rất lớn) gần root.
- H-tree route thường yêu cầu NDR và shielding để bảo vệ SI/EM.

### 10.4 Fishbone Clock Tree

Cấu trúc: sử dụng một clock trunk chính, với nhiều nhánh branch vuông góc tỏa ra hai bên — tương tự hình xương cá (fishbone).

**Ưu điểm**:

- Wire length ngắn hơn H-tree → insertion delay nhỏ hơn.
- Clock capacitance thấp hơn → dynamic power thấp hơn.
- Phù hợp với floorplan hình chữ nhật dài.
- Routing implementation đơn giản hơn H-tree.

**Nhược điểm**:

- Skew lớn hơn H-tree do routing không hoàn toàn đối xứng.
- Nhạy cảm hơn với placement imbalance.
- Timing variation tăng với sink ở xa.
- Scale kém với chip rất lớn.

### 10.5 Multi-Source Clock Tree (MSCTS)

Cấu trúc: kiến trúc distributed — sử dụng nhiều clock injection point (nhiều source) được đồng bộ hóa với nhau. Thay vì một root duy nhất drive toàn chip, nhiều root cùng drive từng vùng khác nhau của chip.

**Ưu điểm**:

- Skew thấp hơn single-source CTS vì mỗi region có source riêng gần với sink hơn.
- OCV tolerance tốt hơn nhờ clock path ngắn hơn trong từng region.
- Power thấp hơn clock mesh.
- Routing resource ít hơn clock mesh.
- Linh hoạt hơn với floorplan có nhiều macro.

**Nhược điểm**:

- Phức tạp hơn về CTS optimization — phải synchronize nhiều source.
- Clock synchronization giữa các source khó hơn — nếu không đồng bộ sẽ gây domain crossing issues.
- Clock balancing challenging hơn.
- Yêu cầu quản lý skew group cẩn thận.
- Debug và phân tích khó hơn.

---

## 11. So sánh tổng hợp các loại Clock Tree

|Tiêu chí|Conventional|Clock Mesh|H-Tree|Fishbone|MSCTS|
|---|---|---|---|---|---|
|Skew|Cao|Rất thấp|Thấp|Trung bình|Thấp|
|Power|Thấp|Rất cao|Trung bình|Thấp|Trung bình|
|Routing complexity|Thấp|Rất cao|Trung bình|Thấp|Trung bình-cao|
|OCV tolerance|Thấp|Rất cao|Tốt|Trung bình|Tốt|
|Clock gating|Dễ|Khó|Trung bình|Dễ|Trung bình|
|Scalability|Kém|Tốt|Tốt|Trung bình|Tốt|
|Debug|Dễ|Khó|Trung bình|Dễ|Khó|

---

## 12. Mối liên kết giữa các khái niệm — Tổng hợp

Toàn bộ các khái niệm trong CTS có mối liên hệ chặt chẽ theo chuỗi nhân quả:

**Skew → Timing**: Skew lớn gây setup hoặc hold violation. CTS phải minimize và balance skew để đảm bảo cả setup slack và hold slack dương.

**Slew → Skew và Delay**: Slew quá lớn tăng propagation delay không đồng đều giữa các branch, làm xấu skew. Đồng thời slew lớn tăng short-circuit power và suy giảm duty cycle.

**Duty Cycle → Pulse Width → Functional Correctness**: Duty cycle degradation dẫn đến pulse width violation, gây functional failure trong Flip-flop không capture được data đúng.

**Crosstalk → Jitter → Uncertainty**: Crosstalk gây thêm jitter trên clock signal. Jitter được mô hình hóa thêm vào clock uncertainty, làm giảm timing margin.

**NDR → Crosstalk và Reliability**: NDR (wide width, large spacing, shielding) giảm crosstalk và EM, từ đó cải thiện jitter và reliability của clock distribution.

**Clock Buffer (vs Regular Buffer) → Duty Cycle → Pulse Width**: Sử dụng clock buffer thay vì regular buffer duy trì balanced rise/fall → duty cycle ổn định → pulse width hợp lệ.

**Clock Gating → Complexity → CTS**: Clock gating giảm power nhưng tăng clock tree complexity, đòi hỏi CTS xử lý ICG cell đúng cách (nonstop pin, clock gating check).

**Exception Pins → Skew Balancing Accuracy**: Định nghĩa đúng stop/nonstop/exclude/float/ignore pin đảm bảo CTS balance đúng những gì cần balance, không lãng phí tài nguyên vào những điểm không quan trọng, và không bỏ sót điểm quan trọng.

**Skew Group → Global vs Local Skew**: Phân chia skew group đúng theo clock domain tách biệt global skew (không kiểm soát được) ra khỏi local skew (phải minimize), giúp CTS converge nhanh hơn và hiệu quả hơn.

