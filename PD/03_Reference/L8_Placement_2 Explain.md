# Power Optimization

## Trang 6: Power Analysis

### 1. Phân loại Tổng Công suất Tiêu thụ (Total Power Dissipation)

- Tổng công suất tiêu thụ của một thiết kế được chia thành hai thành phần chính: Công suất tĩnh (Static Power Dissipation) và Công suất động (Dynamic Power Dissipation).
    
- Trên sơ đồ mạch CMOS, công suất tĩnh liên quan đến dòng điện rò rỉ ($I_{leak}$) và công suất động liên quan đến dòng điện chuyển mạch ($I_{switch}$).
### 2. Định nghĩa Công suất Tĩnh và Công suất Động

- **Static Power (Công suất tĩnh):** Là phép tính toán công suất rò rỉ (leakage power), mức tiêu thụ điện này xảy ra ngay cả khi cell không chuyển đổi trạng thái (not switching).
    
- **Dynamic Power (Công suất động):** Là lượng công suất bị tiêu tán bởi cell do hai nguyên nhân: sự chuyển đổi trạng thái (switching) và hiện tượng đoản mạch (short circuit).
### 3. Quy trình Ước tính Công suất (Power Analysis Flow)

- Việc ước tính sự tiêu tán công suất (tĩnh và động) trong thiết kế được thực hiện qua nhiều giai đoạn trong suốt chu kỳ thiết kế.
    
- Ở giai đoạn Placement, kết quả phân tích có độ chính xác thấp hơn (less accurate).
    
- Ở giai đoạn Post-route, kết quả phân tích có độ chính xác cao hơn do sử dụng dữ liệu hoạt động chuyển đổi đầu vào từ quá trình mô phỏng (simulation input switching activity).

## Trang 7: Components of Power in Standard Cells

### 1. Tổng Công suất (Total Power)

- Phương trình cơ bản: **Total Power = Static Power + Dynamic Power**.
### 2. Công suất Động (Dynamic Power)

Năng lượng tiêu hao khi mạch hoạt động, tín hiệu đầu vào có sự chuyển đổi (switching). Bao gồm 2 thành phần:

- **Switching Power (Công suất chuyển mạch):**
    
    - Là thành phần chiếm tỷ trọng lớn nhất (Dominant).
        
    - Nguồn gốc: Quá trình nạp (charge) và xả (discharge) tụ điện tải ($C_{load}$) khi tín hiệu chuyển đổi giữa '0' và '1'.
        
    - Phương trình: $P_{switching}=\alpha\cdot C\cdot V^{2}\cdot f$ (Phụ thuộc vào hệ số hoạt động $\alpha$, điện dung tải $C$, điện áp nguồn $V$, và tần số $f$).
        
- **Internal Power (Công suất nội bộ):**
    
    - Nguồn gốc: Xảy ra bên trong standard cell, bao gồm chuyển mạch node nội bộ và hiện tượng đoản mạch (Short-circuit).
        
    - Công suất đoản mạch (Short-circuit power): Xảy ra tức thời khi cả transistor P và N cùng bật, tạo đường dẫn trực tiếp từ nguồn ($VDD$) xuống đất ($GND$).
        
    - Phương trình: $P_{short\_circuit}\sim I_{sc}\cdot V\cdot t_{sc}\cdot f$ (Phụ thuộc vào dòng đoản mạch $I_{sc}$, điện áp $V$, thời gian duy trì đoản mạch $t_{sc}$, và tần số $f$).

### 3. Công suất Tĩnh (Static Power / Leakage)

Năng lượng rò rỉ khi mạch ở trạng thái tĩnh (không có chuyển đổi tín hiệu, ngõ vào ổn định ở '0' hoặc '1').

- **Nguồn gốc (4 loại dòng rò rỉ chính):**
    
    1. Dòng phân cực ngược diode (Diode reverse bias current - I1): - Dòng rò rỉ sinh ra tại các tiếp giáp bán dẫn p-n (giữa vùng Drain/Source và phần đế Substrate) khi các tiếp giáp này bị đặt trong trạng thái phân cực ngược.
        
    2. Dòng rỉ dưới ngưỡng (Sub-threshold current - I2): Dòng điện rò rỉ tiếp tục chảy qua kênh dẫn từ Source sang Drain ngay cả khi transistor đã được tắt (điện áp cực cổng $V_{gs}$ nhỏ hơn điện áp ngưỡng $V_{th}$). Đây thường là thành phần chiếm tỷ trọng lớn nhất trong tổng công suất rò rỉ.
        
    3. Dòng rỉ máng do cực cổng cảm ứng (Gate-induced drain leakage - I3): Xảy ra khi có một điện trường rất mạnh sinh ra tại vùng Drain (đoạn nằm dưới mép của cực cổng Gate), khiến các hạt tải điện xuyên hầm (tunneling) trực tiếp từ vùng Drain xuống đế (Substrate).
        
    4. Dòng rỉ qua lớp oxit cổng (Gate oxide leakage - I4): Dòng rò rỉ đi xuyên qua lớp cách điện mỏng (oxit cổng) nằm giữa cực Gate và kênh dẫn bên dưới. Hiện tượng này xảy ra do hiệu ứng xuyên hầm lượng tử khi lớp oxit cổng ngày càng được làm mỏng đi trong các tiến trình công nghệ mới.
        
- **Yếu tố ảnh hưởng:** 
	- Loại điện áp ngưỡng của Standard Cell (HVT, SVT, LVT).
    
    - Nhiệt độ (Temperature).
        
    - Quy trình công nghệ (Process).

- **Phương trình:** $P_{leakage}= V\cdot I_{leak}$ (Điện áp nguồn nhân với tổng dòng rò).

![[Pasted image 20260517170659.png]]
## Trang 8, 9, 10: Leakage vs Dynamic Power

### 1. Xu hướng Công suất theo Tiến trình Công nghệ (Leakage vs Dynamic Power Trend)

- **Sự thay đổi tỷ trọng:** Ở các công nghệ cũ, công suất động (Dynamic Power) chiếm phần lớn. Tuy nhiên, khi thu nhỏ tiến trình công nghệ (ví dụ từ 65nm xuống 16nm), công suất rò rỉ (Leakage Power) gia tăng nhanh chóng và trở thành thành phần chính yếu, tương đương với công suất động.
    
- **Nguyên nhân gia tăng rò rỉ ở các tiến trình nhỏ:**
    
    - Việc giảm điện áp nguồn (supply voltage) yêu cầu phải giảm điện áp ngưỡng ($V_{th}$), hệ quả là làm tăng dòng rò.
        
    - Lớp oxit cổng mỏng hơn (thinner gate oxide) làm tăng dòng xuyên hầm qua cực cổng.
        
    - Các hiệu ứng kênh ngắn (Short-channel effects) trở nên rõ rệt và nổi bật hơn.

### 2. Bản chất các thành phần Công suất Rò rỉ (Leakage Power Components)

Công suất rò rỉ bao gồm các dòng điện chạy ngay cả khi transistor lý tưởng đang ở trạng thái TẮT (OFF). Có 3 thành phần cốt lõi:

- **Subthreshold Leakage Current (Dòng rỉ dưới ngưỡng):** Xảy ra khi điện áp cực cổng thấp hơn điện áp ngưỡng ($V_{th}$). Một sự dẫn điện yếu vẫn tồn tại, cho phép dòng điện chảy từ cực Drain sang cực Source. Đây là yếu tố đóng góp lớn nhất vào rò rỉ trong các công nghệ thu nhỏ.
    
- **Gate Leakage Current (Dòng rỉ qua cực cổng):** Xảy ra do hiện tượng hạt tải điện xuyên hầm qua lớp oxit cổng (gate oxide tunneling) dưới tác động của điện trường mạnh vắt ngang qua lớp oxit này.
    
- **Reverse-Biased Junction Leakage (Dòng rỉ tiếp giáp phân cực ngược):** Xảy ra tại các tiếp giáp P-N phân cực ngược nằm giữa vùng Source/Drain và lớp đế (substrate). Hiện tượng này xuất hiện khi cực Source/Drain của NMOS được nối với mức điện áp $VDD$, hoặc cực Source/Drain của PMOS được nối với mức điện áp $VSS$.
    
- _Lưu ý:_ Sơ đồ minh họa vật lý cũng chỉ ra sự tồn tại của dòng rỉ GIDL và dòng rỉ đâm thủng (Punch through current).
### 3. Các biến số tác động đến Công suất (Power Factors)

**Tác động đến Công suất Rò rỉ (Leakage Power):**

- **Các yếu tố làm tăng rò rỉ:** Giảm điện áp ngưỡng , giảm độ dày lớp oxit , tăng điện áp nguồn , tăng nhiệt độ , tăng chiều rộng Transistor , và áp dụng phân cực đế thuận (Forward Body Bias).
    
- **Các yếu tố làm giảm rò rỉ:** Áp dụng phân cực đế ngược (Reverse Body Bias) và tận dụng hiệu ứng xếp chồng Transistor (Stacking effect) phụ thuộc vào trạng thái logic.
    
- **Yếu tố gây biến thiên:** Sự thay đổi trong quy trình sản xuất (Process Variation) làm mức độ rò rỉ biến thiên không đồng đều trên từng thiết bị.

**Tác động đến Công suất Động (Dynamic Power):**

(Sự gia tăng của các yếu tố dưới đây đều dẫn đến TĂNG công suất động)

- Hệ số chuyển đổi (Switching Activity) và Tần số (Frequency): Tạo ra nhiều lần chuyển trạng thái, làm tăng dòng điện tiêu thụ.
    
- Điện dung tải (Capacitance), Fanout và Chiều dài dây (Wirelength): Tạo ra mức tải lớn hơn, đòi hỏi dòng điện chuyển mạch lớn hơn.
    
- Điện áp nguồn ($V$): Làm dòng điện tăng theo $V$ và công suất tăng theo bình phương điện áp ($V^2$).
    
- Nhiễu (Glitching / Hazards): Sinh ra các quá trình chuyển trạng thái tín hiệu dư thừa, không cần thiết.
    
- Tính song song (Parallelism): Gây ra các đỉnh dòng điện tức thời (instantaneous current spikes) rất lớn.

## Trang 11: Static power analysis

### 1. Thiết lập Dữ liệu Hoạt động Chuyển đổi (Set Switching Activity)

Cung cấp thông số chuyển đổi trạng thái để tính toán công suất động qua 2 phương pháp:

- **Phương pháp ước tính:** Lệnh `set_default_switching_activity -input_activity 0.2 -period 8.0` gán giá trị chuyển đổi mặc định cho toàn thiết kế (tỷ lệ chuyển đổi ngõ vào 0.2, chu kỳ 8.0).
    
- **Phương pháp dữ liệu mô phỏng:** Lệnh `read_fsdb design.fsdb` (kèm `compute_switching_activity`) hoặc `read_saif design.saif` nạp dữ liệu hoạt động thực tế từ tệp mô phỏng để cho kết quả chính xác nhất.
### 2. Thực thi và Xuất báo cáo (Run Power Analysis)

- **Lệnh:** `report_power` và `report_power -hierarchy`.
    
- **Chức năng:** Tính toán và trích xuất bảng tổng kết mức tiêu thụ công suất tổng thể, bao gồm phân tách chi tiết theo cấu trúc phân cấp (hierarchy).
### 3. Phân tích số liệu Báo cáo (Report Analysis)

- **Tỷ trọng các loại Công suất (Total Power Breakdown):**
    
    - **Total Leakage Power:** 83.75% (400.024) - Chiếm tỷ trọng lớn nhất, tiêu hao ngay cả khi mạch không chuyển đổi trạng thái.
        
    - **Total Internal Power:** 15.12% (72.234).
        
    - **Total Switching Power:** 1.11% (5.334) - Chiếm tỷ trọng rất nhỏ.
        
    - **Total Power:** 477.593 (Tổng của 3 thành phần trên).
        
- **Phân bổ theo nhóm phần tử (Group Analysis):**
    
    - **Macro:** Nhóm tiêu thụ năng lượng lớn nhất, chiếm 95.04% tổng công suất (453.9), phần lớn đến từ hao phí rò rỉ (Leakage).
        
    - **Sequential (Tuần tự):** Chiếm 3.60%.
        
    - **Combinational (Tổ hợp):** Chiếm 1.33%.
        
    - **IO và Clock:** Tiêu thụ không đáng kể (xấp xỉ 0%).

## Trang 12, 13: Power Optimization

### 1. Nguyên tắc cốt lõi (quan trọng nhất để hiểu toàn bộ phần này)

Power Optimization trong post-placement **không bao giờ được phép làm hỏng Timing**. Đây là ràng buộc bất biến — tool chỉ thực hiện các thay đổi khi chứng minh được rằng Timing không bị vi phạm.
### 2. Hai loại power được tối ưu và lệnh điều chỉnh tỉ lệ

Tool tối ưu đồng thời hai thành phần:
- **Leakage power** (Static): tiêu hao ngay cả khi Cell không switching — giảm bằng cách dùng Cell có Vt cao hơn.
- **Dynamic power**: tiêu hao do switching — giảm bằng cách rút ngắn wirelength, bỏ Buffer/Inverter thừa, giảm drive strength.

Lệnh thiết lập tỉ lệ ưu tiên giữa hai loại:
```tcl
set_db opt_leakage_to_dynamic_ratio 0.5
```
Giá trị `0.5` = cân bằng ngang nhau giữa leakage và dynamic. Giá trị gần `1.0` → ưu tiên leakage; gần `0.0` → ưu tiên dynamic.
### 3. Năm hành động cụ thể tool thực hiện

| Hành động                                   | Mục tiêu                              | Loại power được giảm |
| ------------------------------------------- | ------------------------------------- | -------------------- |
| Swap Cell sang variant tiêu thụ thấp hơn    | Giảm leakage                          | Static               |
| Dùng Cell có drive strength nhỏ hơn         | Giảm area và leakage                  | Static + Dynamic     |
| Chọn Cell có Vt cao hơn (higher-VT)         | Giảm subthreshold leakage             | Static               |
| Xóa Buffer/Inverter không cần thiết         | Giảm số lượng Node switching          | Dynamic              |
| Điều chỉnh vị trí Cell để tối ưu wirelength | Giảm capacitance tải → giảm P = αCV²f | Dynamic              |

Tất cả các thay đổi đều phải tuân thủ UPF power budget constraints.
### 4. Quy trình setup và chạy lệnh (theo thứ tự thực thi)

**Bước 1 — Annotate switching activity** (để tool tính Dynamic power chính xác):
```tcl
set_default_switching_activity -global_activity 0.2 -sequential_activity 0.8
```
- `global_activity 0.2`: 20% signal trên design switching mỗi clock cycle.
- `sequential_activity 0.8`: Flip-flop switching 80% thời gian.
- Nếu có file mô phỏng thực tế: dùng `read_fsdb` hoặc `read_saif` thay thế để chính xác hơn.

**Bước 2 — Thiết lập tỉ lệ leakage/dynamic:**
```tcl
set_db opt_leakage_to_dynamic_ratio 0.5
```
**Bước 3 — Chạy power optimization:**
```tcl
opt_power -pre_cts
```
Flag `-pre_cts` xác định đây là giai đoạn trước Clock Tree Synthesis — timing lúc này dùng ideal clock, chưa có actual clock skew.
### 5. Điều kiện tiên quyết (prerequisite)

Để tối ưu leakage power hiệu quả thông qua VT swapping, **thư viện phải chứa multi-VT cells** (HVT, SVT, LVT cho cùng một Cell type). Nếu thư viện chỉ có một VT, tool không thể thực hiện VT swapping — đây là yêu cầu từ phía setup môi trường, không phải lệnh.
### Tóm tắt luồng logic

```
Setup switching activity
        ↓
Set leakage/dynamic ratio
        ↓
opt_power -pre_cts
        ↓
Tool tự động: VT swap + drive strength reduction
              + buffer removal + wirelength optimization
        ↓
Kết quả: power giảm, timing KHÔNG đổi
```

## Trang 14: Standard Cell VT 

### 1. Khái niệm cốt lõi: VT là gì và tại sao quan trọng

VT (Threshold Voltage) là điện áp ngưỡng để transistor bắt đầu dẫn điện. Đây là thông số quyết định **trade-off trực tiếp giữa Performance và Power** của một Standard Cell. Toàn bộ chiến lược Power Optimization dựa trên việc swap VT giữa các Cell.
### 2. Ba loại VT và đặc tính

| VT Type                   | Performance                              | Power                                  | Ghi chú                     |
| ------------------------- | ---------------------------------------- | -------------------------------------- | --------------------------- |
| Low VT (LVT)              | Cao nhất — nhanh nhất                    | Cao nhất — tiêu thụ nhiều nhất         | Dùng trên critical path     |
| Standard/Regular VT (SVT) | Trung bình — nhanh hơn HVT, chậm hơn LVT | Trung bình — nhiều hơn HVT, ít hơn LVT | Mặc định                    |
| High VT (HVT)             | Thấp nhất — chậm nhất                    | Thấp nhất — tiêu thụ ít nhất           | Dùng trên non-critical path |
Quan hệ: `LVT: delay=1ns, power=7nW` → `SVT: delay=2ns, power=6nW` → `HVT: delay=3ns, power=4nW` (số liệu minh họa từ slide).

### 3. Điểm then chốt nhất: All VT cells có cùng footprint

**Tất cả các biến thể VT của cùng một Cell type đều có:**
- Cùng kích thước vật lý (same size)
- Cùng vị trí Pin
**Hệ quả trực tiếp:** Khi tool thực hiện VT swapping (ví dụ: đổi một Inverter từ LVT sang HVT), **không có bất kỳ tác động nào đến Placement hoặc Routing**. Đây là lý do VT swapping là kỹ thuật Power Optimization hiệu quả nhất — chi phí thực thi bằng không về mặt layout.
### 4. Ứng dụng trong Physical Design flow

Chiến lược phân bổ VT theo Timing Slack:
- Cell nằm trên **critical path** (Slack = 0 hoặc âm) → phải dùng LVT để đảm bảo Timing.
- Cell nằm trên **non-critical path** (Slack dương, nhiều margin) → tool swap sang HVT để tiết kiệm leakage power.
- Mục tiêu cuối: **maximize số lượng HVT cells** trong design mà không vi phạm bất kỳ Timing constraint nào.
### 5. Lệnh kiểm tra phân bố VT trong design
```tcl
report_threshold_voltage_group
```
Lệnh này report số lượng Cell thuộc từng VT group (HVT/SVT/LVT) hiện có trong design — dùng để đánh giá kết quả sau `opt_power`.
### Tóm tắt logic quan hệ
```
Vt thấp → transistor dẫn dễ hơn → switching nhanh hơn
                                  → leakage current cao hơn

Vt cao  → transistor dẫn khó hơn → switching chậm hơn
                                  → leakage current thấp hơn

Footprint giống nhau → swap VT = zero layout cost
```
## Trang 15: Clock Gating Techniques

### 1. Vấn đề cần giải quyết: Tại sao cần Clock Gating

Dynamic power tiêu thụ theo công thức P = αCV²f. Trong đó, clock network là thành phần switching **liên tục 100% thời gian** (activity factor α = 1), đồng thời có wirelength lớn nhất trong toàn bộ design → clock network là nguồn tiêu thụ dynamic power lớn nhất. Clock Gating giải quyết vấn đề này bằng cách **ngắt clock đến các Flip-flop khi chúng không cần capture data**.
### 2. Traditional Clock Gating — AND Gate (phương pháp cũ, có vấn đề)

**Cơ chế:** Clock được AND với tín hiệu Enable.
```
CLK ──┐
      AND ── Out (Gated Clock)
EN  ──┘
```
**Vấn đề nghiêm trọng:** Nếu tín hiệu EN thay đổi trong khi CLK đang ở mức HIGH → output của AND gate tạo ra **glitch** (xung giả ngắn). Glitch này lan truyền vào clock network → gây unnecessary switching → tăng dynamic power thay vì giảm. Đây là lý do Traditional Clock Gating không được dùng trong thiết kế hiện đại.
### 3. Integrated Clock Gating (ICG) Cell — Giải pháp chuẩn trong thiết kế hiện đại ^ICG-cell

**Cấu trúc bên trong ICG Cell:**
```
EN ── Latch (negative-edge triggered) ── AND ── Out (Gated Clock)
CLK ─────────────────────────────────────┘
```
**Nguyên lý hoạt động:**
- Latch capture tín hiệu EN khi CLK = LOW (negative phase).
- Output của Latch chỉ thay đổi khi CLK thấp → tín hiệu EN đã ổn định trước khi CLK lên HIGH.
- AND gate nhận EN đã ổn định → **không thể tạo glitch**.
**Kết quả:** Clock output sạch, không có xung giả, đảm bảo Timing integrity.
### 4. So sánh trực tiếp waveform

|Điểm so sánh|Traditional (AND)|ICG Cell|
|---|---|---|
|Glitch khi EN thay đổi|Có — xuất hiện xung giả|Không — EN được latch trước|
|An toàn với Timing|Không đảm bảo|Đảm bảo|
|Mức độ phức tạp|Đơn giản|Phức tạp hơn nhưng chuẩn|
|Dùng trong thiết kế hiện đại|Không|Có — ưu tiên dùng|
### 5. Lợi ích Power và ứng dụng thực tế

ICG Cell được insert tự động trong quá trình Logic Synthesis (Genus) với mục tiêu giảm dynamic power của clock network. Sau Placement, ICG cells được đặt với constraint `set_dont_touch` để tool không di chuyển hoặc optimize chúng tùy tiện (xem lại slide 3: `set_dont_touch icg_reg_* true`).

**Hiệu quả thực tế từ slide 48:** Clock Gating Efficiency đạt **81.2%** — tức 81.2% Flip-flop trong design được gating, tiết kiệm ước tính **28.6% clock power**.
### 6. Lệnh kiểm tra
```tcl
report_clock_gating
```
Report này cho biết: tổng số sequential cells, số cells được clock-gated, số cells không được gated, clock gating efficiency (%), và ước tính power savings.
### Tóm tắt logic

```
Clock switching liên tục → Dynamic power cao
        ↓
Traditional AND gate → glitch → không dùng
        ↓
ICG Cell (Latch + AND) → EN ổn định trước CLK rising edge
        ↓
Gated clock sạch → giảm dynamic power không vi phạm Timing
        ↓
report_clock_gating → kiểm tra hiệu quả
```
## Trang 16: Power Optimization Log 

#### Nội dung log
Log hiển thị quá trình chạy `opt_power -pre_cts` thông qua engine **PowerOpt** của Innovus.

Bảng trong log theo dõi 3 metric theo từng iteration:

- **Density**: mật độ Placement (55.86% — không thay đổi, đúng như kỳ vọng vì VT swap không ảnh hưởng layout)
- **WNS / TNS**: đều giữ nguyên 0.000 → Timing không bị vi phạm trong suốt quá trình optimize
- **Commits**: số lần tool commit thay đổi Cell (61 lần, sau đó 26 lần)

Dòng cuối xác nhận tool hoàn thành trong ~12 giây thực tế.
#### Bốn optimization engines của Cadence Innovus

|Engine|Chức năng|
|---|---|
|**GigaPlace**|Placement|
|**GigaOpt**|Timing/Power Optimization|
|**NanoRoute**|Routing|
|**CCOpt**|Clock Tree Synthesis|
|**PowerOpt**|Power Optimization|
#### Điểm quan trọng cần nhớ
WNS và TNS bằng 0 trong suốt quá trình = tool **không bao giờ trade Timing để đổi lấy Power** — đúng với nguyên tắc đã nêu ở slide 12.

# Area Optimization
Trang 18, 19
### 1. Nguyên tắc cốt lõi
Area Optimization hoạt động theo ràng buộc bất biến giống Power Optimization: **chỉ tác động lên các Net có positive Slack** — tức là chỉ những path có timing margin dư mới được phép optimize. Điều này đảm bảo tool không bao giờ tạo ra DRV violation mới hoặc làm hỏng Timing trong quá trình giảm area.
### 2. Hai hành động cụ thể tool thực hiện

**Hành động 1 — Downsizing:** Thay thế Cell đang dùng bằng variant có drive strength nhỏ hơn trong cùng Cell type.
```
INV8X  →  INV4X
```
Cell nhỏ hơn → chiếm ít diện tích silicon hơn → area giảm. Chỉ thực hiện khi path đó có đủ Slack để chịu được delay tăng thêm do drive strength giảm.

**Hành động 2 — Buffer/Inverter removal:** Xóa các Buffer hoặc Inverter được insert trong quá trình Synthesis hoặc HFNS nhưng sau Placement không còn cần thiết nữa (wirelength thực tế ngắn hơn dự đoán → không cần drive mạnh → Buffer thừa).
### 3. Thời điểm thực thi trong flow

Area Optimization **tự động chạy sau Timing Optimization** theo mặc định — không cần lệnh riêng. Tuy nhiên có thể gọi tường minh:
```tcl
opt_design -pre_cts -area
```
Flag `-area` chỉ định tool tập trung vào area reclaim sau khi Timing đã được giải quyết.
### 4. Lệnh report và đọc kết quả (Slide 19)
```tcl
report_area -summary      # tổng area toàn design
report_area -hierarchy    # breakdown theo hierarchy module
report_gate_count         # số lượng gate
```
**Đọc log Area Reclaim Optimization:**
```
Density: 54.33% → 54.31%   # density giảm nhẹ = area được thu hồi
WNS: 0.000                  # Timing không bị ảnh hưởng
TNS: 0.000
Commits: 57                 # 57 lần tool commit thay đổi Cell
```
**Đọc report_area -summary:**
```
Hinst Name    Module Name    Inst Count    Total Area
leon                         34690         390396.650
```
- **Inst Count**: tổng số instance (Cell) trong design.
- **Total Area**: tổng diện tích tính bằng đơn vị của process node (thường là µm²).
### 5. Quan hệ giữa Area Optimization và Power Optimization

Hai tác dụng phụ có lợi khi area giảm:
- Downsizing → drive strength nhỏ hơn → leakage current giảm → **leakage power giảm**.
- Xóa Buffer → ít Node switching hơn → **dynamic power giảm**.
Area Optimization vì vậy đồng thời cải thiện cả Power — đây là lý do trong flow, Area và Power Optimization được nhóm chung thành một giai đoạn post-placement.
### Tóm tắt logic
```
Sau Timing Optimization → Slack dương tồn tại trên nhiều path
        ↓
opt_design -pre_cts -area
        ↓
Tool xác định path có positive Slack
        ↓
Downsizing (INV8X → INV4X) + Buffer/Inverter removal
        ↓
Area giảm, Leakage giảm, Dynamic power giảm
Timing KHÔNG thay đổi, DRV KHÔNG vi phạm
        ↓
report_area -summary / -hierarchy / report_gate_count
```

# Tie Cells, Spare Cells, MBFF
## [[Tie cells]]

## [[Spare cells]]

## [[MBFF]] (Multi-Bit Flip Flop)

# Congestion

## Trang 29, 30: Nền tảng lý thuyết Congestion

### 1. Định nghĩa Congestion và tại sao nó quan trọng ^Congestion1

Congestion (hay routing congestion) xảy ra khi **số routing track cần thiết để route các Net tại một vùng vượt quá số routing track có sẵn** tại vùng đó. Đây là vấn đề về routing resource — không phải vấn đề về logic hay Timing.

**Hệ quả trực tiếp:** Nếu congestion không được giải quyết tại Placement stage → Routing stage sẽ không thể complete → design không thể Tape-out. Đây là lý do congestion phải được kiểm soát **trước khi chạy Routing**.

Định nghĩa định lượng:
```
Horizontal Congestion = Horizontal available routing tracks
                      - Required horizontal routing tracks

Vertical Congestion   = Vertical available routing tracks
                      - Required vertical routing tracks
```
**Quy tắc đọc giá trị:**
- Giá trị âm → congested (demand vượt supply)
- Giá trị = 0 → vừa đủ
- Giá trị dương → còn dư routing resource

→ **Số càng thấp (càng âm) = congestion càng nghiêm trọng.**

### 2. GCELL — Đơn vị cơ bản của Congestion analysis ^Gcell1

**GCELL (Global routing Cell)** là đơn vị không gian cơ bản mà tool dùng để phân tích congestion. Tool chia toàn bộ Core area thành một **lưới GCELL** — mỗi GCELL là một ô hình chữ nhật chứa một số routing track cố định theo chiều ngang (horizontal) và chiều dọc (vertical).

Mỗi GCELL lưu trữ hai thông số:
- **Available tracks**: số routing track thực sự có trên metal layer trong GCELL đó.
- **Required tracks**: số routing track tool ước tính cần để route tất cả Net đi qua GCELL đó.

Congestion được tính và báo cáo **theo từng GCELL** — không phải theo toàn bộ design → cho phép xác định chính xác vùng nào bị congested.

**Overflow** xảy ra khi required tracks > available tracks tại một GCELL cụ thể — GCELL đó được đánh dấu là overflow.

### 3. Global Routing — Cơ chế tool phát hiện Congestion (Trang 30)

**Global Routing trong context Placement không phải là Routing thực sự** — đây là bước **ước tính routing (Early Routing Estimation)** — tool chỉ phân bổ Net vào các GCELL một cách gần đúng để đánh giá routing demand, không tạo ra actual wire.

**Các chức năng cụ thể của Global Routing tại Placement stage:**
- Ước tính tổng wirelength của toàn bộ design.
- Xác định routing demand (số track cần) tại mỗi GCELL.
- Phát hiện các GCELL có nguy cơ overflow.
- Phân bổ ước tính horizontal và vertical routing resource theo từng GCELL.

**Thông tin từ hình minh họa slide 30:**
- GCELL grid 4×4 với kích thước mỗi GCELL ví dụ: 24 × 28 (đơn vị [[3.2. LEF - Library Exchange Format#^track1|track]]).
- Con số này thể hiện số track available theo chiều ngang và dọc trong một GCELL.

**Lệnh gọi Global Routing tường minh:**
```tcl
route_early_global
```
Lệnh này thường được chạy **sau** `place_opt_design` để có ước tính congestion chính xác hơn dựa trên Placement thực tế. Kết quả của `route_early_global` được dùng làm input cho `report_congestion`.

**Lợi ích quan trọng:** Thông tin từ Global Routing được **feed ngược lại vào Placement engine** → Placement engine điều chỉnh vị trí Cell để tránh tạo routing hotspot → cải thiện cả PPA và routability đồng thời.

### 4. Nguyên nhân phổ biến nhất gây Congestion

Từ slide 29: **Congestion thường xuất hiện tại vùng có cell density cao.** Lý do:
- Cell density cao → nhiều Pin cần được kết nối trong một vùng nhỏ.
- Nhiều Net phải đi qua cùng một vùng → routing demand tăng đột biến tại các GCELL trong vùng đó.
- Available tracks không tăng theo (fixed bởi technology và metal layer definition).
- → Overflow xuất hiện.
### Tóm tắt logic
```
Core area được chia thành lưới GCELL
        ↓
Mỗi GCELL có: available tracks (fixed) vs required tracks (depends on Placement)
        ↓
route_early_global
        ↓
Tool ước tính routing demand tại từng GCELL
        ↓
Required > Available tại GCELL nào → GCELL đó bị Overflow
        ↓
Horizontal Congestion = Available H - Required H
Vertical Congestion   = Available V - Required V
(giá trị âm = congested)
        ↓
Thông tin này feed ngược vào Placement engine
→ Placer tránh tạo hotspot
→ PPA và routability cải thiện
```
## Trang 31, 32, 33: Đo lường và đánh giá Congestion 

### 1. Hai metric đo Congestion và ngưỡng pass/fail (quan trọng nhất)

Có hai metric độc lập để đánh giá congestion — cần hiểu rõ sự khác biệt giữa chúng:

**Metric 1 — Overflow Score (Global Congestion):**
- Đo **tổng thể** routing demand vs available tracks trên toàn design.
- Tính theo tỉ lệ % số GCELL bị overflow.
- Range: 0% → 100%.
- **Ngưỡng pass/fail: Overflow > 1% → design khó route.**
```
Overflow: 6191 = 1302 (0.72% H) + 4889 (2.76% V)
```
Trong ví dụ này: Horizontal overflow = 0.72% (acceptable), Vertical overflow = 2.76% (vượt ngưỡng 1% → có vấn đề).

**Metric 2 — Hotspot Score (Local Congestion):**
- Đo **mức độ nghiêm trọng** của các vùng congested cục bộ — tập trung vào worst-case areas.
- Không phải tỉ lệ % mà là điểm số normalized.
- **Ngưỡng pass/fail: Hotspot score > 100 → typically not routable.**
```
normalized max hotspot: 119.80   → vùng tệ nhất = 119.80 (vượt ngưỡng 100)
normalized total hotspot: 320.00
```
**Tại sao cần cả hai metric:**
- Overflow thấp nhưng Hotspot cao → design tổng thể ổn nhưng có một vài điểm cực kỳ tắc nghẽn cục bộ → vẫn không route được.
- Overflow cao nhưng Hotspot thấp → congestion phân bố đều → dễ fix hơn.
- Cần cả hai đều pass mới coi design là routable.

### 2. Lệnh đầy đủ cho Congestion Analysis (Trang 31)
```tcl
# Bước 1: Chạy early global routing để có dữ liệu congestion
route_early_global

# Bước 2: Report overflow (global congestion)
report_congestion -overflow

# Bước 3: Report hotspot (local congestion)
report_congestion -hotspot

# Bước 4: Visualize trên GUI
gui_show_congestion

# Bước 5: Report density map
report_density_map
```
Thứ tự thực thi bắt buộc: `route_early_global` phải chạy trước các lệnh report — nếu không có dữ liệu global routing thì report không có dữ liệu để tính.

### 3. Đọc chi tiết report_congestion -overflow (Trang 32)
```
Usage: (18.4%H 27.5%V) = (7.999e+05um 7.463e+05um) = (467798 400940)
Overflow: 6191 = 1302 (0.72% H) + 4889 (2.76% V)

Congestion distribution:
Remain  cntH          cntV
-5:     32   0.02%    397   0.22%
-4:     63   0.03%    187   0.11%
-3:    142   0.08%    402   0.23%
-2:    268   0.15%    669   0.38%
-1:    474   0.26%   1821   1.03%
 0:   1123   0.62%   3854   2.17%
 1:   2134   1.18%   6829   3.85%
 2:   3264   1.81%  10776   6.08%
...
 5: 160036  88.63% 122646  69.17%
```
**Cách đọc bảng Congestion distribution:**
- **Remain column**: giá trị = Available tracks - Required tracks tại mỗi GCELL.
- **Giá trị âm** (Remain = -1, -2, -3...): GCELL bị overflow — càng âm càng tắc nghẽn nghiêm trọng.
- **Giá trị = 0**: GCELL vừa đủ track, không overflow nhưng không có margin.
- **Giá trị dương**: GCELL còn dư track — tốt.
- **cntH / cntV**: số lượng GCELL và tỉ lệ % theo chiều ngang/dọc ở mỗi mức Remain.

Trong ví dụ: 88.63% GCELL horizontal có Remain = 5 (rất thoải mái), nhưng vẫn có GCELL với Remain = -5 (cực kỳ tắc nghẽn).
### 4. Đọc chi tiết report_congestion -hotspot (Trang 32)
```
[hotspot] top 5 congestion hotspot bounding boxes:
[hotspot]  top  |    hotspot bbox              | hotspot score
[hotspot]   1   | 274.66 438.90 329.38 575.70  |   117.25
[hotspot]   2   | 589.30 452.58 644.02 562.02  |    83.34
[hotspot]   3   | 343.06 274.74 438.82 315.78  |    70.03
[hotspot]   4   |  69.46 233.70  96.82 261.06  |    13.38
[hotspot]   5   | 329.38 507.30 356.74 534.66  |     3.87
```
**Cách đọc:**
- **hotspot bbox**: tọa độ {x1 y1 x2 y2} của vùng congested — dùng để locate chính xác trên layout GUI.
- **hotspot score**: mức độ nghiêm trọng — score #1 = 117.25 > 100 → **vùng này không routable**, cần fix.
- Score #2 = 83.34 < 100 → chưa critical nhưng cần theo dõi.
- Tool luôn report top 5 worst hotspots — ưu tiên fix từ score cao nhất xuống.

### 5. Congestion Map visualization và ý nghĩa màu sắc (Trang 33)

Congestion Map hiển thị trực quan mức độ congestion trên toàn layout:

|Màu|Ý nghĩa|
|---|---|
|Đỏ / Hồng đậm|Congestion nghiêm trọng nhất — overflow lớn|
|Cam / Vàng|Congestion trung bình — cần chú ý|
|Xanh lá|Congestion nhẹ — acceptable|
|Xanh dương / Tối|Không congested — routing resource dư|
**Nguyên tắc quan trọng từ slide 33:** Phải minimize congestion tại **Placement stage** để maximize design routability — càng để sang Routing stage mới xử lý thì chi phí fix càng cao (có thể phải re-Place toàn bộ vùng đó).

Ba hình trên slide 33 minh họa quá trình cải thiện:
```
Congested → Less congested → Least congested
```
Vùng khoanh tròn trắng (hotspot chính) giảm dần từ đỏ sang xanh qua các iteration optimize.
### 6. GUI commands bổ sung
```tcl
gui_show_congestion    # bật hiển thị Congestion Map trong GUI
report_density_map     # report density distribution — 
                       # phân biệt với congestion: 
                       # density = mật độ Cell, 
                       # congestion = routing resource shortage
```

**Phân biệt Density Map vs Congestion Map:**
- **Density Map**: cho biết vùng nào có nhiều Cell → input để dự đoán congestion.
- **Congestion Map**: cho biết vùng nào thực sự thiếu routing track → kết quả thực tế sau global routing.
- Density cao không nhất thiết dẫn đến congestion nếu các Cell có ít Pin và Net ngắn.
### Tóm tắt logic
```
route_early_global (tạo dữ liệu)
        ↓
report_congestion -overflow     → Global metric: % GCELL overflow
                                   Pass: < 1% H và V
report_congestion -hotspot      → Local metric: worst-case score
                                   Pass: max score < 100
        ↓
Overflow > 1% hoặc Hotspot > 100 → cần fix
        ↓
gui_show_congestion             → locate vùng cần fix trên layout
report_density_map              → identify nguyên nhân density cao
        ↓
Iterate fix → re-run route_early_global → re-check metrics
        ↓
Cả hai metrics pass → ready for Routing stage
```
## Trang 34, 35, 36, 37: Kỹ thuật fix Congestion trực tiếp tại Placement 

### 1. Nguyên nhân gây Congestion (Trang 34) — Phải biết nguyên nhân trước khi fix

Bảy nguyên nhân từ slide 34, phân thành hai nhóm:

**Nhóm Layout/Placement:**
- High Standard Cell density trong một vùng giới hạn.
- High Standard Cell Pin density (nhiều Pin tập trung → nhiều Net phải route qua cùng một vùng).
- Standard Cells đặt gần Macro — Net phải detour quanh Macro.
- High Pin density tại cạnh của Macro — tất cả Net vào/ra Macro tập trung tại một vùng hẹp.

**Nhóm Design/Flow:**
- Poor Floorplan — Macro đặt sai vị trí tạo ra chướng ngại vật.
- Poor Synthesis — Netlist tạo ra quá nhiều Net cross-chip.
- Poor PG Grid strategy — quá nhiều PG stripe chiếm routing track.

**Ý nghĩa thực tế:** Mỗi nguyên nhân tương ứng với một kỹ thuật fix khác nhau. Phần 3 này cover các kỹ thuật fix **trực tiếp tại Placement** (nhóm đầu). Phần 4 cover nhóm sau.
### 2. Tổng quan bảy kỹ thuật fix Congestion (Trang 34)

| Kỹ thuật                                       | Phần nào cover    |
| ---------------------------------------------- | ----------------- |
| Add Placement Blockages (hard/partial/soft)    | Phần 3 — Trang 36 |
| Add Halo around Macros/memories                | Phần 3 — Trang 36 |
| Add Cell Padding                               | Phần 3 — Trang 37 |
| Change Placement strategy to congestion-driven | Phần 4 — Trang 38 |
| Update PG Grid                                 | Phần 4 — Trang 39 |
| Update Floorplan                               | Phần 4 — Trang 40 |
| Update Synthesis to be physically-aware        | Phần 4 — Trang 41 |
### 3. Phân loại Placement Blockages — Bốn loại và cơ chế hoạt động (Trang 35, 36)

Placement Blockage là vùng được định nghĩa trong layout mà tool **không được** (hoặc **bị giới hạn**) đặt Cell vào. Cơ chế fix congestion: buộc tool spread Cell sang vùng khác → giảm density tại vùng congested → giảm routing demand.

**Loại 1 — Hard Blockage:**
- Ngăn hoàn toàn mọi Cell khỏi vùng được chỉ định.
- Dùng khi: vùng đó không được phép có bất kỳ Standard Cell nào (ví dụ: vùng dưới Macro, vùng reserved cho analog).
```tcl
create_placement_blockage \
    -type hard \
    -boundary {10 10 50 50}
# hoặc dùng tọa độ thực tế:
# create_place_blockage -area 516.33950 288.29300 532.47700 301.35650 -type hard
```
**Loại 2 — Partial Blockage:**
- Cho phép Placement nhưng giới hạn tối đa một tỉ lệ % utilization trong vùng đó.
- Dùng khi: muốn giảm density nhưng vẫn cho phép một số Cell để tránh wasted area.
```tcl
create_placement_blockage \
    -area 10 10 50 50 \
    -type partial \
    -density 50        # tối đa 50% utilization trong vùng này
```
**Loại 3 — Soft Blockage:**

- Chỉ cho phép một số loại Cell nhất định (Buffer, Inverter, Clock gater) vào vùng đó.
- Dùng khi: muốn giữ vùng cho Clock Tree buffer hoặc optimization cells, không cho logic cells thông thường.
```tcl
create_placement_blockage \
    -area 10 10 50 50 \
    -type soft \
    -density 25        # chỉ 25% utilization, chỉ dành cho Buffer/Inverter
```
**Loại 4 — Macro-only Blockage:**
- Ngăn Placement của Macro trong vùng đó, nhưng vẫn cho phép Standard Cell.
- Dùng khi: cần kiểm soát vị trí Macro trong Floorplan.

### 4. Routing Blockages — Khái niệm và phân biệt với Placement Blockage (Trang 35)

Routing Blockage là vùng được định nghĩa mà **router không được route Net** trên các metal layer được chỉ định trong vùng đó.

**Mục đích:**
- Bảo vệ vùng nhạy cảm (analog, clock, power).
- Preserve routing resource cho các Net quan trọng.
- Enforce design constraints cụ thể.
```tcl
create_routing_blockage \
    -layers {M2 M3 M4} \
    -boundary {100 100 200 200}
```
**Phân biệt quan trọng:**

| |Placement Blockage|Routing Blockage|
|---|---|---|
|Tác động|Ngăn đặt Cell|Ngăn route Net|
|Layer|Không áp dụng|Chỉ định metal layer cụ thể|
|Mục tiêu|Giảm cell density|Bảo vệ routing resource|
```tcl
report_placement_blockages    # kiểm tra tất cả Placement Blockage
report_routing_blockages      # kiểm tra tất cả Routing Blockage
delete_placement_blockage <name>  # xóa Blockage nếu cần
```
### 5. Halo — Kỹ thuật tạo vùng keep-out quanh Macro (Trang 35, 36)

Halo là một dạng Placement Blockage đặc biệt — tự động tạo vùng keep-out **bao quanh Macro hoặc Memory** với khoảng cách cố định. Không có Cell nào được phép đặt trong vùng Halo.

**Tại sao cần Halo:**
- Standard Cells đặt quá gần Macro → tất cả Net vào/ra Macro phải route trong không gian cực hẹp → Pin density cao → congestion nghiêm trọng tại cạnh Macro.
- Halo tạo ra buffer zone → Net có không gian để spread ra → giảm routing density tại cạnh Macro.
```tcl
# Tạo Halo cho một Macro cụ thể
create_place_halo \
    -halo_deltas {17 17 17 17} \    # {left bottom right top} tính bằng micron
    -insts ROM                       # tên Macro instance

# Tạo Halo cho tất cả Macro trong design
create_place_halo \
    -halo_deltas {4 4 4 4} \
    -all_blocks
```
`-halo_deltas {left bottom right top}`: khoảng cách Halo theo bốn hướng tính từ cạnh Macro. Giá trị thường dùng: 4–20 micron tùy kích thước Macro và mức độ congestion.

### 6. Cell Padding — Kỹ thuật tạo spacing constraint cho từng Cell type (Trang 37)

Cell Padding thêm spacing constraint **xung quanh một Cell type cụ thể** — tool phải để khoảng trống theo số track được chỉ định xung quanh mỗi instance của Cell type đó trong quá trình Placement.

**Khi nào dùng Cell Padding:**
- Flip-flop, MBFF, complex gate có nhiều Pin → khi đặt sát nhau → Pin density cực cao → routing giữa các Pin của các Cell liền kề bị tắc nghẽn.
- Cell Padding buộc tool giữ khoảng cách tối thiểu → routing có room để đi.
```tcl
# Set Cell Padding cho SDFFQX2
set_cell_padding \
    -cell SDFFQX2 \
    -right_side 2 \     # 2 track padding bên phải
    -top_side 3 \       # 3 track padding phía trên
    -left_side 2 \      # 2 track padding bên trái
    -bottom_side 5      # 5 track padding phía dưới
# Report để verify
report_cell_padding
# Xóa padding nếu cần
remove_cell_padding -cell SDFFQX2
```
**Đọc report_cell_padding:**
```
Cell Name    left    right    top    bottom
SDFFQX2       2        2       3        5
Total 1 cells have padding.
```
**Đơn vị của padding:** số **routing track** (không phải micron) — phụ thuộc vào [[3.2. LEF - Library Exchange Format#^track1|track]] pitch của technology node.

**Trade-off:** Cell Padding tăng area (vì mỗi Cell chiếm thêm không gian) nhưng giảm local routing congestion — chỉ dùng cho Cell type có nhiều Pin, không dùng đại trà cho tất cả Cell.

### 7. Workflow tổng thể khi fix congestion tại Placement
```tcl
# 1. Identify vùng congested
route_early_global
report_congestion -hotspot      # lấy tọa độ bbox của hotspot

# 2. Fix theo từng kỹ thuật (có thể kết hợp)

# Fix Macro-related congestion
create_place_halo -halo_deltas {4 4 4 4} -all_blocks

# Fix high-density area congestion
create_placement_blockage -area <x1 y1 x2 y2> -type partial -density 50

# Fix Pin density congestion của specific Cell type
set_cell_padding -cell SDFFQX2 -left_side 2 -right_side 2

# 3. Re-place và verify
refine_place
legalize_placement
check_place

# 4. Re-check congestion
route_early_global
report_congestion -overflow
report_congestion -hotspot
```
### Tóm tắt so sánh ba kỹ thuật

|Kỹ thuật|Cơ chế|Dùng khi|Lệnh chính|
|---|---|---|---|
|Placement Blockage|Cấm/giới hạn Cell trong vùng|Cell density quá cao tại vùng cụ thể|`create_placement_blockage`|
|Halo|Keep-out zone quanh Macro|Congestion tại cạnh Macro/Memory|`create_place_halo`|
|Cell Padding|Spacing quanh Cell type cụ thể|Pin density cao của FF/complex gate|`set_cell_padding`|
Ba kỹ thuật này tác động **trực tiếp vào Placement** — hiệu quả ngay lập tức, không cần thay đổi Netlist hay Floorplan. Đây là bước fix đầu tiên nên thử trước khi escalate lên các giải pháp cấp cao hơn.

## Trang 38, 39, 40, 41 - Chiến lược fix Congestion cấp cao

### 1. Vị trí của phần này trong hierarchy fix Congestion

Bốn kỹ thuật trong phần này được dùng khi các kỹ thuật trực tiếp tại Placement (Blockage, Halo, Cell Padding) **không đủ để giải quyết congestion** — hoặc khi nguyên nhân gốc rễ nằm ở cấp độ cao hơn: strategy, power grid, floorplan, hoặc synthesis. Chi phí thực thi tăng dần theo thứ tự từ trang 38 → 41.
### 2. Congestion-Driven Placement Strategy — Trang 38

**Vấn đề cốt lõi: Timing-driven vs Congestion-driven**

Mặc định, Placement engine hoạt động theo chế độ **Timing-driven**: tool tối ưu Placement để minimize wirelength → các Cell liên quan được đặt gần nhau nhất có thể → wirelength ngắn → Timing tốt. Tuy nhiên, khi quá nhiều Cell cluster lại một vùng → routing demand tại vùng đó vượt available tracks → congestion.

Minh họa từ slide 38:
```
Timing-driven:                    Congestion-driven:
A B C D (gần nhau)               D B C A (spread ra)
E F G H                          E F G H
Channel Density: 3               Channel Density: 2
(track:3) → Unroutable           (track:2) → Routable
Wire length: ngắn hơn            Wire length: dài hơn
```
Timing-driven tạo wirelength ngắn hơn nhưng channel density = 3 vượt quá capacity = 2 → **Unroutable**. Congestion-driven spread Cell ra → channel density = 2 = capacity → **Routable** dù wirelength dài hơn.

**Điều kiện áp dụng:** Design phải có **đủ Timing margin (positive Slack)** để chịu được delay tăng thêm do wirelength dài hơn. Nếu Timing đã tight → không thể dùng congestion-driven.

**Lệnh:**
```tcl
# Bật congestion-driven mode
set_db place_global_cong_effort high

# Spread Cell đều khắp design thay vì cluster
set_db place_global_uniform_density true

# Re-run Placement với strategy mới
place_opt_design

# Verify kết quả
route_early_global
report_congestion -hotspot
```
**Trade-off rõ ràng:**
- Wirelength tăng → Timing có thể bị ảnh hưởng → phải re-check Timing sau khi switch sang congestion-driven.
- Sau `place_opt_design` với congestion-driven → chạy `time_design -pre_cts` để confirm Timing vẫn acceptable.
### 3. Power Grid Adjustment — Trang 39

**Cơ chế gây congestion từ PG Grid:**

PG (Power/Ground) Grid gồm các metal stripe chạy ngang và dọc trên nhiều metal layer để phân phối VDD/VSS. Mỗi PG stripe **chiếm một số routing track** trên metal layer đó → các Net signal không thể sử dụng track đó → available tracks cho signal routing giảm. Nếu PG Grid quá dày → routing resource cho signal bị squeeze → congestion.

**Giải pháp:**
- Giảm số lượng PG stripe tại vùng congested — merge hai stripe gần nhau thành một stripe rộng hơn.
- Kết quả: giải phóng routing track cho signal Net → congestion giảm.

**Ràng buộc bắt buộc:** PG Grid sau khi điều chỉnh vẫn phải đáp ứng **EM (Electromigration) và IR-drop requirements** — không được giảm PG stripe đến mức voltage drop quá lớn hoặc current density vượt ngưỡng EM.

**Concept IR-Drop-Aware Placement:** Tool có thể thực hiện Placement với awareness về IR-drop map — đặt Cell tiêu thụ nhiều power gần các PG stripe → giảm IR-drop. Hai lệnh liên quan:
```tcl
analyze_rail          # phân tích IR-drop và EM trên PG network
report_power_grid     # report trạng thái PG Grid hiện tại
```
**Lưu ý quan trọng:** Power Grid Adjustment yêu cầu quay lại **Floorplan stage** để modify PG Grid → sau đó re-run Placement. Đây là lý do chi phí fix cao hơn Phần 3.

### 4. Floorplan Adjustment — Trang 40

**Khi nào congestion có nguyên nhân từ Floorplan:**

Nếu Macro được đặt sai vị trí → tạo ra các vùng mà Net phải đi vòng quanh Macro → routing path dài hơn → nhiều Net cross qua cùng một kênh hẹp → congestion. Đây là **structural congestion** — không thể fix bằng Blockage hay Cell Padding vì nguyên nhân là topology của Floorplan.

**Minh họa từ slide 40:**
```
Before:                          After:
[Macro][Macro]                   [Macro]  [Macro]
[congested regions]   →          
[Macro][Macro]                   [Macro]  [Macro]
                                 (spread ra, có channel rõ ràng)
```
Các vùng khoanh đỏ (congested regions) biến mất sau khi Macro được re-arrange để tạo routing channel rõ ràng hơn giữa chúng.

**Khi nào phải restart Floorplan:**
- Congestion hotspot xuất hiện **nhất quán** tại cùng một vùng qua nhiều iteration Placement fix.
- Vùng congested nằm ngay cạnh hoặc giữa các Macro.
- Tất cả kỹ thuật Phần 3 đã áp dụng nhưng hotspot score vẫn > 100.

**Đây là fix tốn kém nhất trong Physical Design flow** — phải quay lại Floorplan, di chuyển Macro, re-run toàn bộ Placement pipeline. Tuy nhiên đôi khi không có lựa chọn khác.
### 5. Physically-Aware Synthesis — Trang 41

**Vấn đề của Traditional Synthesis Flow:**

Trong Traditional Synthesis, tool Synthesis (Genus) chỉ biết về RTL và Timing constraints — **không biết gì về physical layout**. Kết quả: Synthesis tối ưu logic mà không xét đến việc các Cell sẽ được đặt ở đâu trong layout → có thể tạo ra Netlist với nhiều long-range Net → khi Placement, các Net này buộc phải cross qua nhiều GCELL → routing demand cao → congestion.
```
Traditional Flow:
RTL + Constraints + Timing Libs
        ↓
Synthesis (không biết layout)
        ↓
Netlist → Physical Design
(Floorplan data chỉ được dùng một chiều, không feedback)
```
**Physically-Aware Synthesis Flow:**
Tool Synthesis nhận thêm **DEF file** chứa Floorplan/blockage/placement data ngay từ đầu → Synthesis biết vị trí Macro, boundary, blockage → tối ưu logic với awareness về physical constraints → Netlist output có ít long-range Net hơn → congestion giảm từ gốc.
```
Physically-Aware Flow:
RTL + Constraints + Timing Libs + DEF (Floorplan data)
        ↓
Synthesis (biết layout)
        ↓
Netlist → Physical Design
(DEF feedback loop: Placement data feed ngược lại Synthesis)
```
**Lệnh trong Genus (Synthesis tool — không phải Innovus):**
```tcl
# Đọc Floorplan DEF vào Synthesis tool
read_def floorplan.def

# Bật physically-aware mode
set_db physical_design_mode true

# Tăng congestion effort trong Synthesis
set_db opt_congestion_effort high

# Chạy Synthesis với physical awareness
syn_generic    # generic synthesis
syn_map        # technology mapping
syn_opt        # optimization
```
**Kết quả:** Netlist từ Physically-Aware Synthesis có wirelength tổng ngắn hơn, ít cross-chip Net hơn → khi Placement, routing demand phân bố đều hơn → congestion giảm ngay từ đầu flow.

**Đây là giải pháp tốt nhất về dài hạn** nhưng yêu cầu re-run toàn bộ Synthesis → Physical Design pipeline → chi phí thời gian lớn nhất.

### 6. Thứ tự ưu tiên áp dụng các kỹ thuật fix
```
Phát hiện congestion (route_early_global + report_congestion)
        ↓
Bước 1 — Thử Phần 3 trước (chi phí thấp nhất):
  → Placement Blockage / Halo / Cell Padding
  → Re-check: nếu pass → done
        ↓
Bước 2 — Congestion-Driven Placement (Trang 38):
  → set_db place_global_cong_effort high
  → set_db place_global_uniform_density true
  → place_opt_design → re-check Timing và Congestion
        ↓
Bước 3 — Power Grid Adjustment (Trang 39):
  → analyze_rail → modify PG Grid → re-run Placement
  → Đảm bảo EM/IR requirements vẫn met
        ↓
Bước 4 — Floorplan Adjustment (Trang 40):
  → Re-arrange Macro → re-run toàn bộ Placement pipeline
        ↓
Bước 5 — Physically-Aware Synthesis (Trang 41):
  → Re-run Synthesis với DEF → re-run toàn bộ PD flow
  → Giải quyết congestion từ gốc
```
### Tóm tắt so sánh bốn kỹ thuật

|Kỹ thuật|Cơ chế|Điều kiện áp dụng|Chi phí|Lệnh chính|
|---|---|---|---|---|
|Congestion-Driven Placement|Spread Cell để giảm local density|Design có đủ Timing margin|Thấp — re-run Placement|`set_db place_global_cong_effort high` + `place_opt_design`|
|PG Grid Adjustment|Giảm PG stripe → giải phóng routing track|Congestion do PG Grid dày|Trung bình — modify Floorplan PG|`analyze_rail` + `report_power_grid`|
|Floorplan Adjustment|Re-arrange Macro → tạo routing channel|Congestion do Macro placement|Cao — restart Floorplan|Manual Macro re-placement|
|Physically-Aware Synthesis|Synthesis biết layout → ít long-range Net|Congestion có nguyên nhân từ Synthesis|Rất cao — re-run toàn bộ flow|`read_def` + `set_db physical_design_mode true` + `syn_*`|
# Outputs and Results Evaluation
Trang 44 đến 50

## Outputs and Results Evaluation — Trang 44 đến 50

### 1. Tư duy tổng quát: Placement được coi là "Done" khi nào?

Đây là câu hỏi quan trọng nhất cần trả lời trước khi học từng tiêu chí cụ thể. Placement stage kết thúc và design được phép chuyển sang CTS khi **đồng thời thỏa mãn tất cả** sáu điều kiện sau (từ slide 50):
```
✓ All Cells Legally Placed
✓ Routable (minimal congestion)
✓ No DRV Violations
✓ No EMIR Violations
✓ PPA Requirements Met
✓ Ready for CTS
```
Không có ngoại lệ — thiếu bất kỳ điều kiện nào → phải quay lại iterate. Đây là **exit criteria /kraɪˈtɪr.i.ə/** của Placement stage.

### 2. Tiêu chí 1 — Placement Quality: All Cells Legally Placed (Trang 46)

**Ý nghĩa "legally placed":**

Một Cell được gọi là legally placed khi thỏa mãn đồng thời:
- **Không overlap**: không có Cell nào chồng lên Cell khác dù chỉ một phần.
- **Snap to grid**: Cell phải nằm đúng trên placement grid (multiple của site width).
- **Không có unplaced Cell**: mọi Cell trong Netlist đều có tọa độ xác định trong layout.
- **Orientation hợp lệ**: Cell được flip/rotate đúng theo row orientation.

**Tại sao quan trọng:** Nếu Cell overlap hoặc không snap to grid → Routing không thể complete vì router giả định Cell ở vị trí hợp lệ. Unplaced Cell → Net của Cell đó hoàn toàn không được route → design sai chức năng.

**Cách verify:** Chạy `check_place` — nếu output báo `Unplaced = 0` và không có overlap violation → Placement Quality pass.

Ví dụ từ slide 46:
```
Placed = 35595    (Fixed = 4)
Unplaced = 0
Placement Density: 46.46%
```
`Unplaced = 0` → tất cả Cell đã có vị trí. `Fixed = 4` là các Cell được fix cứng (thường là IO hoặc special cells) — không phải lỗi.

### 3. Tiêu chí 2 — Timing & DRV: Không có (hoặc rất ít) vi phạm (Trang 45)

**Tại sao Timing phải được check tại Placement stage:**

Tại Placement stage, clock vẫn là **ideal clock** (chưa có CTS). Timing analysis lúc này gọi là **Pre-CTS timing**. Đây là ước tính — sau CTS sẽ có clock latency và skew thực tế làm Timing thay đổi. Tuy nhiên nếu Pre-CTS Timing đã vi phạm nghiêm trọng → Post-CTS Timing sẽ còn tệ hơn → không thể sign-off.

**Ngưỡng chấp nhận được:**
- **WNS (Worst Negative Slack)**: nên = 0 hoặc rất nhỏ âm (minor violations). Từ slide 45: WNS = 0.001 ns → acceptable.
- **TNS (Total Negative Slack)**: tổng tất cả Slack âm → nên = 0. Slide 45: TNS = 0.000 → clean.
- **Violating Paths**: số path vi phạm → lý tưởng = 0. Slide 45: 0 violating paths.

**Ba loại DRV phải được clean tại Placement (trừ clock Net):**

|DRV Type|Ý nghĩa|Tại sao trừ clock Net|
|---|---|---|
|Max fanout|Một output drive quá nhiều inputs|Clock Net có fanout rất lớn — sẽ được fix bởi CTS|
|Max capacitance|Net có tổng capacitance vượt ngưỡng|Clock Net sẽ được buffer bởi CTS|
|Max transition|Slew rate quá chậm (signal rise/fall time quá dài)|CTS sẽ insert buffer để fix|
**Lý do trừ clock Net:** Tại Pre-CTS stage, clock Net chưa được synthesize → clock Net còn là ideal wire nối thẳng từ source đến tất cả Flip-flop → fanout cực lớn, capacitance cực cao → DRV vi phạm là expected và sẽ được giải quyết hoàn toàn bởi CTS. Nếu fix clock Net DRV tại đây → CTS sẽ bị confuse.
### 4. Tiêu chí 3 — Routability: Congestion đạt ngưỡng (Trang 46)

Hai ngưỡng pass/fail đã được giải thích chi tiết trong Phần 2, nhắc lại để hoàn chỉnh:

|Metric|Ngưỡng Pass|Ý nghĩa khi fail|
|---|---|---|
|Global Congestion (Overflow)|< 1%|Design khó route — router sẽ tạo DRC violations|
|Local Congestion (Hotspot score)|< 100|Vùng đó typically không routable|
Từ slide 46, ví dụ design đạt:
```
Overflow: 2 = 2 (0.00% H) + 0 (0.00% V)   → 0.00% << 1% ✓
Hotspot normalized: 0.00                    → 0.00 << 100 ✓
```
Design này routable.
### 5. Tiêu chí 4 — Power: Đáp ứng power budget và không có EMIR (Trang 47)

**Power consumption phải trong giới hạn:**

Tổng Power = Dynamic Power (Internal + Switching) + Leakage Power. Từ ví dụ slide 47:
```
Total Internal Power:   72.64 mW   (15.19%)
Total Switching Power:   5.55 mW    (1.16%)
Total Leakage Power:   400.01 mW   (83.64%)
Total Power:           478.20 mW
```
Leakage chiếm 83.64% tổng power — điển hình cho advanced node. Số liệu này phải được so sánh với **power budget** được định nghĩa trong design spec để xác định pass/fail.

**EMIR (Electromigration và IR-drop) — Tại sao nguy hiểm:**

**IR-drop:** Khi current chạy qua resistance của PG wire → voltage tại điểm xa power source thấp hơn VDD danh nghĩa. Nếu IR-drop quá lớn → VDD thực tế tại Cell quá thấp → Cell hoạt động chậm hơn (delay tăng) → Timing vi phạm → chip fail trong operation dù Timing analysis nói pass.

**Electromigration (EM):** Khi current density trong metal wire quá cao → metal atoms bị di chuyển dần theo chiều dòng điện → wire bị mỏng dần → đứt wire sau một thời gian hoạt động → reliability failure. EM là vấn đề long-term — chip có thể hoạt động đúng lúc mới xuất xưởng nhưng fail sau vài năm.

**Early IR-drop analysis tại Placement:** Đây là **static IR-drop analysis** — chưa phải final (final analysis sẽ chạy sau Routing). Mục đích: phát hiện sớm các vùng PG Grid yếu → điều chỉnh PG Grid hoặc di chuyển high-power Cell trước khi route.

Hình trong slide 47 cho thấy IR-drop map với color scale — vùng đỏ = IR-drop cao nhất (gần 14.84 mV) → cần điều chỉnh.

### 6. Tiêu chí 5 — Clock Network: Sẵn sàng cho CTS (Trang 48)

Tại Placement stage, clock chưa được synthesize nhưng **phải verify các thông số clock sẽ ảnh hưởng đến CTS**:

**Các thông số cần kiểm tra:**
```
Clock Name    Period    Waveform         Source       Attributes
CLK           2.000     {0.000 1.000}    clk_port     propagated
SCAN_CLK     10.000     {0.000 5.000}    scan_clk     propagated
CORE_CLK      1.250     {0.000 0.625}    pll_clk      generated
```
- Ba clock domain: CLK (2ns period), SCAN_CLK (10ns), CORE_CLK (1.25ns — fastest, generated từ PLL).
- Tất cả là propagated clock → CTS sẽ synthesize clock tree từ các source này.

**Worst Skew / Max Latency tại Pre-CTS (ideal clock):**
```
CLK:      Worst Skew = 0.045ns, Max Latency = 0.312ns
CORE_CLK: Worst Skew = 0.038ns, Max Latency = 0.287ns
SCAN_CLK: Worst Skew = 0.102ns, Max Latency = 0.611ns
```
Đây là Skew và Latency **lý tưởng** (ideal clock) — sau CTS, Skew sẽ tăng lên do clock tree có finite delay. CTS target thường là Skew < 100–200 ps tùy design.

**Clock Gating summary:**
```
Total Sequential Cells:        124,580
Clock Gated Sequential Cells:  101,220   (81.2% efficiency)
Ungated Sequential Cells:       23,360
Estimated Clock Power Savings:  28.6%
```
81.2% cells được clock-gated → 28.6% clock power được tiết kiệm. Đây là metric quan trọng confirm rằng clock gating strategy đã được implement đúng trong Synthesis.

### 7. Outputs được tạo ra tại cuối Placement (Trang 44)

Ba file output chính:

**place.db** — Database file chứa toàn bộ trạng thái design sau Placement (Netlist, Placement coordinates, constraints, timing data). Đây là file dùng để tiếp tục sang CTS stage.

**placement.def** — DEF file chứa physical layout information: vị trí chính xác của từng Cell, Macro, blockage. File này có thể được dùng để feed vào Physically-Aware Synthesis (như đã học ở Phần 4).

**placement.v** — Netlist Verilog sau khi đã được optimize (có thể khác với Netlist đầu vào do cell swapping, buffer insertion/removal, MBFF banking, tie cell insertion, spare cell insertion).
### 8. Placement Summary — Bức tranh tổng thể (Trang 50)

Toàn bộ Placement stage là một **iterative optimization loop** gồm hai nhóm hoạt động song song:

**Nhóm bên trái — Sequential steps:**
```
Set Up Placement Strategies
        ↓
HFNS, Scan Reordering
        ↓
DRV Fixing
        ↓
Timing Optimization
```
**Nhóm bên phải — Post-placement optimization:**
```
Power/Area Optimization
        ↓
Tie Cell, Spare Cell Insertion
        ↓
MBFF Optimization
        ↓
Congestion Optimization
```
Hai nhóm này không nhất thiết chạy tuần tự một lần — có thể iterate nhiều lần (mũi tên vòng lặp trong slide 50) với các trade-off:
- Timing vs Congestion vs Power: fix Timing có thể làm tăng congestion; fix congestion có thể ảnh hưởng Timing.
- Area vs Timing: downsizing giảm area nhưng tăng delay.
- Power vs Performance: VT swap lên HVT giảm power nhưng chậm hơn.

**Placement trong context của toàn bộ PD flow:**
```
Data Import
    ↓
Floorplan
    ↓
Placement  ← Chúng ta đang ở đây
    ↓
CTS
    ↓
Routing
    ↓
Data Export
```
Placement là stage **trung tâm** của PD flow — nhận kết quả từ Floorplan và Synthesis, quyết định chất lượng của CTS và Routing. Một Placement tốt làm cho CTS và Routing dễ dàng hơn nhiều; một Placement kém sẽ tạo ra cascading problems qua tất cả các stage sau.

### Tóm tắt exit criteria theo thứ tự kiểm tra
```
1. check_place → Unplaced = 0, no overlaps
        ↓
2. time_design -pre_cts → WNS ≈ 0, TNS = 0, DRV clean (except clock nets)
        ↓
3. report_congestion -overflow → < 1%
   report_congestion -hotspot  → < 100
        ↓
4. report_power → within budget
   analyze_rail → no critical IR-drop or EM violations
        ↓
5. report_clocks → all clocks defined correctly
   report_clock_gating → gating efficiency acceptable
        ↓
Tất cả pass → write_db placement.db → Ready for CTS
Bất kỳ fail → iterate với kỹ thuật phù hợp → re-check
```
# [[Key Commands]]