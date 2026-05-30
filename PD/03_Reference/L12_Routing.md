# Lecture 12 – Routing (Part II): Phân Tích Lý Thuyết Toàn Diện

**Nguồn:** Lecture 12 – Routing (Part II), Phil Hoang, Tresemi (xây dựng trên tài liệu gốc của Thien Phan)  
**Liên kết:** Lecture 11 – Routing (Part I)

---

## Vị Trí Trong Routing Flow Tổng Thể

L11 đã trình bày nửa đầu của Routing Flow:

```
Setup → Global Routing → Detailed Routing
```

L12 tiếp nối từ điểm đó, phủ phần còn lại:

```
Post-Route Optimization → Chip Finishing & DFM → Post-Route Signoff → Tape-out
```

Toàn bộ flow hoàn chỉnh của Routing gồm năm giai đoạn tuần tự:

```
Setup
  ↓
Global Routing   — Lập kế hoạch topology, layer assignment, tối ưu congestion/timing/routability
  ↓
Detailed Routing — Tạo wire và via hợp lệ, giải quyết DRC/pin access/antenna/timing/SI
  ↓
Post-Route Optimization   ← L12 bắt đầu từ đây
  ↓
Chip Finishing & Signoff
  ↓
Tape-out
```

---

## Phần 1: Tại Sao Cần Post-Route Optimization?

### 1.1 Vấn Đề Căn Bản: Estimated vs. Actual Parasitics

Đây là nền tảng lý thuyết quan trọng nhất để hiểu toàn bộ giai đoạn này.

**Trước routing** (trong CTS và Placement), timing analysis sử dụng **estimated parasitic RC** — tool ước lượng R và C của wire dựa trên wire length dự kiến, không có wire thực tế. Đây là xấp xỉ thống kê, không phải giá trị vật lý thực.

**Sau routing**, `extract_rc` tính toán parasitic thực từ geometry của từng wire và via đã được route. Đây là **actual parasitic** — giá trị ground truth cho implementation flow.

Khoảng cách giữa hai bộ số này phát sinh từ nhiều nguyên nhân:

- Wire bị forced đi vòng do congestion → chiều dài thực dài hơn ước lượng
- Số lượng via thực tế nhiều hơn dự kiến → thêm $R_{via}$ và $C_{via}$
- Coupling capacitance từ wire kề nhau (không có trong estimated model đơn giản)
- SI/noise effect từ aggressor net (như đã phân tích ở L11, Section 14)

Hệ quả: **RC thực sau routing thường lớn hơn RC ước lượng**, dẫn đến:

$$\text{Delay}_{actual} > \text{Delay}_{estimated} \Rightarrow \text{Slack}_{actual} < \text{Slack}_{estimated}$$

Setup violation mới xuất hiện. Transition (Slew) xấu đi dẫn đến DRV (Design Rule Violation) về transition và capacitance. Hold violation cũng có thể xuất hiện do clock skew thay đổi khi wire RC của clock net bị ảnh hưởng.

### 1.2 Mục Tiêu Của Post-Route Optimization

Post-route optimization thực hiện ba nhóm mục tiêu theo thứ tự ưu tiên:

**Nhóm 1 — Timing Closure:** Fix residual setup, hold, và DRV violation sử dụng RC thực.

**Nhóm 2 — Power và Area Recovery:** Sau khi timing đã close, thu hồi power và area đã bị "lãng phí" trong quá trình aggressive optimization ở các giai đoạn trước.

**Nhóm 3 — Chip Finishing (DFM):** Hoàn thiện physical layout để đáp ứng foundry manufacturability requirement.

Nguyên tắc cơ bản: **Các thay đổi phải được localized và minimal-impact** để không làm xáo trộn routing quality vốn đã được tối ưu trong detailed routing.

---

## Phần 2: Post-Route Flow — Kiến Trúc Tổng Thể

### 2.1 Sơ Đồ Flow Và Mối Quan Hệ Giữa Các Giai Đoạn

Post-route flow gồm ba giai đoạn tuần tự, mỗi giai đoạn có điều kiện tiên quyết từ giai đoạn trước:

```
[Routing: Global + Detailed]
              ↓
╔══════════════════════════════════════════════════╗
║            POST-ROUTE OPTIMIZATION               ║
║                                                  ║
║  [Timing Closure]                                ║
║   Setup/Hold fix, DRV fix                        ║
║        ↓                                         ║
║  [Power and Area Recovery]                       ║
║   VT swap, cell sizing, leakage optimization     ║
╚══════════════════════════════════════════════════╝
              ↓
[Chip Finishing and Signoff]
 Filler, redundant via, metal fill, ERC fixing
              ↓
[Post-Route Signoff]
              ↓
[Tape-out]
```

Lưu ý quan trọng: Timing Closure và Power/Area Recovery **đều nằm trong vòng Post-Route Optimization**. Chúng không độc lập hoàn toàn mà có thể iterate nếu recovery làm xuất hiện violation mới.

### 2.2 Điều Kiện Để Chuyển Giai Đoạn

- **Routing → Post-Route Optimization**: Không có open/short, DRC violation ở mức manageable, timing violation trong phạm vi có thể recover
- **Timing Closure → Power/Area Recovery**: Timing clean (hoặc chấp nhận được), không còn DRV
- **Post-Route Optimization → Chip Finishing**: Timing và DRC đã stable
- **Chip Finishing → Signoff**: Tất cả DFM step hoàn thành
- **Signoff → Tape-out**: Tất cả signoff check clean (hoặc waiver có justification)

---

## Phần 3: Timing Closure

### 3.1 Ba Loại Violation Cần Giải Quyết

Timing closure post-route nhắm đến ba nhóm violation:

| Loại | Cơ chế | Hướng fix |
|------|--------|-----------|
| **Setup violation** | $T_{data\_path} > T_{period} - T_{setup} - T_{skew}$ | Giảm delay trên critical path |
| **Hold violation** | $T_{data\_path} < T_{hold} + T_{skew}$ | Tăng delay trên fast path |
| **DRV** (Design Rule Violation về timing) | Transition/slew vượt giới hạn LIB, load capacitance quá lớn | Resize driver, split net |

**DRV** là loại violation không phải layout DRC mà là timing rule: nếu slew của một net quá lớn (tức là tín hiệu chuyển trạng thái quá chậm), cell ở downstream sẽ không hoạt động đúng và có thể gây thêm delay không mong muốn. LIB file định nghĩa `max_transition` và `max_capacitance` cho từng cell — vi phạm các giới hạn này là DRV.

### 3.2 Chiến Lược Timing Closure: Manual Iteration Loop

Với các thiết kế phức tạp, tool tự động không đủ — cần engineer-guided cleanup. Slide trình bày một chiến lược iteration có cấu trúc:

**Bước 1 — Over-constrain early (Siết chặt constraint trước):**

Đặt transition và capacitance margin chặt hơn so với actual requirement để tạo timing guardband. Guardband là buffer margin giúp đảm bảo design vẫn đúng khi có variation (OCV, temperature drift).

Lý do: Nếu chỉ target exact requirement, variation nhỏ cũng đủ gây fail. Over-constraining buộc tool fix violations sâu hơn, tạo ra margin thực sự.

**Bước 2 — Close baseline timing:**

Fix timing và DRV violation hiệu quả mà không cần full signoff sau mỗi iteration. Dùng in-tool STA (nhanh) thay vì signoff-quality STA (chậm) để iterate nhanh hơn.

**Bước 3 — Enable SI/crosstalk timing:**

Bật crosstalk-aware timing analysis và fix các path nhạy cảm với noise. Đây là bước thực hiện sau baseline vì SI analysis nặng hơn về runtime — không hợp lý khi bật ngay từ đầu khi còn nhiều basic violation.

Liên kết với L11 Section 14: SI analysis cần biết coupling capacitance $C_m$ giữa các wire kề nhau. Sau detailed routing, $C_m$ thực tế đã được extract — lúc này SI analysis mới chính xác.

**Bước 4 — Apply targeted ECOs:**

Resize, buffer, reroute, hoặc shield **chỉ tại các điểm cần thiết**. Không thực hiện thay đổi toàn cục.

**Bước 5 — Recover PPA:**

Relax các temporary constraint đã set ở Bước 1, chạy power/area recovery trong khi giữ timing margin.

**Bước 6 — Final signoff loop:**

Recheck timing, SI, DRC, antenna, và EM/IR bằng signoff-quality tool cho đến khi tất cả clean.

### 3.3 Vòng Lặp Cốt Lõi

Vòng lặp cốt lõi của post-route timing closure:

```
Extract RC từ actual wire
      ↓
Check timing (setup, hold, DRV, SI)
      ↓
Fix violations (buffering, resizing, rerouting)
      ↓
Verify fix không tạo violation mới
      ↓
Lặp lại đến khi clean
```

### 3.4 Post-Route Timing Optimization: Kỹ Thuật Chi Tiết

Mục tiêu của post-route timing optimization là refine timing, signal integrity, EM, và DRC violation **sử dụng RC thực tế đã extract**. Mọi thay đổi phải được localized và minimal-impact để preserve routing quality.

Hai mục tiêu chính:
- Fix residual setup và hold violation
- Resolve remaining DRC từ routing

**Kỹ thuật 1: Debug với full clock path (`-path_type full_clock`)**

Khi analyze timing violation, việc expand full clock path (bao gồm cả launch clock path và capture clock path) là bắt buộc để xác định root cause chính xác. Lý do:

- Setup violation có thể do data path chậm **hoặc** clock path không cân bằng (capture clock đến quá sớm)
- Hold violation có thể do data path nhanh **hoặc** launch clock đến muộn hơn capture clock
- Chỉ nhìn vào data path mà không thấy clock path → chẩn đoán sai → fix sai

Formulas:

**Setup check** (trên toàn bộ path):
$$\text{Slack}_{setup} = (T_{capture\_clock\_arrival} + T_{period} - T_{setup}) - (T_{launch\_clock\_arrival} + T_{data\_path})$$

**Hold check:**
$$\text{Slack}_{hold} = (T_{launch\_clock\_arrival} + T_{data\_path}) - (T_{capture\_clock\_arrival} + T_{hold})$$

Nếu $\text{Slack} < 0$ → violation.

**Kỹ thuật 2: Selective Re-Buffering và Cell Re-Sizing**

Đây là kỹ thuật chủ đạo trong post-route timing optimization. Nguyên lý:

**Khi nào upsize (tăng drive strength):**

- **Setup violation**: Cell quá nhỏ, drive strength thấp → RC delay của output net cao → upsize giảm $R_{driver}$ → net delay giảm → slack tăng
- **Transition violation**: Slew quá lớn do driver yếu không drive nổi load capacitance lớn → upsize tăng drive current → slew cải thiện
- **Capacitance violation**: Net load quá lớn so với driver capacity → upsize tăng khả năng drive load

Liên hệ điện tử: Drive strength của cell tỷ lệ với $W/L$ của transistor. Upsize = tăng $W$ → tăng $I_{drain}$ → tăng dòng charge/discharge → $\frac{dV}{dt}$ lớn hơn → transition nhanh hơn → delay giảm.

**Khi nào downsize (giảm drive strength):**

- **Hold violation**: Cell quá lớn, drive strength cao → delay quá nhỏ → data đến capture FF quá sớm → hold fail → downsize tăng $R_{driver}$ → tăng delay → hold margin tăng

**Key considerations:**

- **Balance drive strength vs. leakage power**: Cell lớn hơn → leakage tăng theo $W$ → power penalty. Upsize chỉ khi thực sự cần thiết.
- **Tránh DRC spacing violation khi swap cell lớn hơn**: Nếu neighboring cell không có đủ không gian, tool phải legalize lại → có thể gây ra cascade placement change → ảnh hưởng routing xung quanh.
- **Corner-aware sizing**: Cell được chọn phải đáp ứng timing ở tất cả corners (SS/TT/FF, temperature extremes), không chỉ tại corner đang fix.
- **Apply selectively**: Chỉ thay đổi cell trên violating path. Thay đổi không cần thiết trên clean path có thể gây timing regression.

**Kỹ thuật 3: Critical Net Re-Routing**

Một số net sau detailed routing đi qua path dài hoặc có nhiều via không cần thiết do congestion tại thời điểm routing. Post-route có thể reroute các net này:

- Route trên layer cao hơn (resistivity thấp hơn, ít congested hơn)
- Giảm số via trên path critical (mỗi via có $R_{via}$ và $C_{via}$ đóng góp vào delay)
- Chọn path ngắn hơn nếu không gian đã được giải phóng sau các thay đổi placement

$$\text{Wire delay} \approx 0.38 \cdot R_{wire} \cdot C_{wire} = 0.38 \cdot \left(\rho \frac{L}{W \cdot T}\right) \cdot (C_{lateral} + C_{vertical} + C_{fringe}) \cdot L$$

Trong đó $\rho$ là điện trở suất kim loại, $L$ là chiều dài wire, $W$ là chiều rộng, $T$ là độ dày. Công thức này cho thấy wire delay tỷ lệ với $L^2$ → giảm wire length có tác động rất lớn.

**Kỹ thuật 4: Useful Skew Adjustment**

Thay vì fix timing bằng cách thay đổi data path, điều chỉnh **thời điểm clock đến** tại từng Flip-flop để tạo ra slack có lợi. Kỹ thuật nâng cao:

- Trên path có setup violation: Delay clock tại capture FF (clock đến muộn hơn) → tăng required time → slack tăng
- Trên path có hold violation: Advance clock tại launch FF (clock đến sớm hơn) → tăng launch time → data cũng phải đến sớm hơn → hold margin tăng

Tuy nhiên, useful skew phải được cân bằng toàn cục — skewing clock tại một FF có thể ảnh hưởng các path khác qua cùng FF đó.

---

## Phần 4: ECO — Engineering Change Order

### 4.1 Định Nghĩa Và Vai Trò

ECO (Engineering Change Order) là cơ chế thực hiện **thay đổi cục bộ, có mục tiêu cụ thể** vào design late in the flow (sau routing) với **tác động nhỏ nhất có thể** đến phần còn lại của implementation.

Mục đích: Fix remaining timing violation mà không trigger lại full place-and-route flow — vì full re-run tốn nhiều thời gian (có thể nhiều giờ đến nhiều ngày) và có nguy cơ introduce regression trên các path đã clean.

Nguyên tắc cốt lõi của ECO: **Thay đổi nhỏ, localized, và minimally disruptive.**

### 4.2 Phân Biệt Hai Loại ECO

**Timing ECO** (nội dung L12): Fix timing violation bằng cell resize, swap, insert, delete — không thay đổi logical function của design.

**Functional ECO**: Thay đổi logic function (fix bug phát hiện muộn). Phức tạp hơn nhiều vì cần thêm/xóa logic gate, và phải pass LEC sau đó.

L12 chỉ đề cập timing ECO.

### 4.3 Các Fix ECO Điển Hình

| Loại Fix | Mục Đích | Kỹ Thuật |
|----------|----------|----------|
| Resize/swap cell | Fix setup hoặc hold | Thay cell bằng version lớn hơn/nhỏ hơn cùng function |
| Insert buffer | Fix setup violation | Buffer thêm drive strength cho net dài |
| Downsize cell | Fix hold violation | Cell nhỏ hơn → delay lớn hơn → hold margin tăng |
| Delete cell | Remove redundant logic từ earlier optimization | Giảm area và leakage |
| Add net + connect pin | Kết nối logic mới sau insert cell | Necessary after cell insertion |

### 4.4 ECO Flow: Vòng Lặp STA ↔ PnR

ECO không phải là one-shot operation mà là **iterative loop** giữa STA tool và PnR tool:

```
STA tool phân tích → xác định violating path
      ↓
STA tool generate ECO command (resize U1234, insert buffer tại net_X, ...)
      ↓
ECO command file được import vào PnR tool
      ↓
PnR tool thực thi ECO:
  - eco_add_cell: Insert cell mới tại location cụ thể
  - eco_resize_cell: Thay cell bằng version khác size
  - eco_swap_cell: Thay cell bằng cell cùng function khác drive strength
  - eco_delete_cell: Xóa cell không cần thiết
  - eco_add_net: Thêm net mới
  - eco_connect_pin: Kết nối pin của cell với net
      ↓
refine_placement với eco_mode = true → legalize placement thay đổi
      ↓
route_eco: Chỉ reroute các net bị ảnh hưởng bởi ECO change, giữ nguyên phần còn lại
      ↓
Re-run STA → verify fix và check for new violation
      ↓
Lặp lại đến khi all violations resolved hoặc no further improvement
```

### 4.5 ECO Routing: Nguyên Lý Giảm Thiểu Tác Động

ECO routing **khác hoàn toàn** với routing thông thường về triết lý:

- **Chỉ route lại các net bị modify bởi ECO** — không touch các net không liên quan
- **Preserve existing routing tối đa** — giữ nguyên wire của net không thay đổi
- **Incremental routing** thay vì full reroute — chỉ route đoạn wire mới/thay đổi
- **Legalize chỉ cell bị ảnh hưởng** — không re-legalize toàn bộ placement
- **Minimize timing regression** — bất kỳ net nào bị touch đều có nguy cơ thay đổi RC → potential regression

Lý do triết lý này quan trọng: Ở giai đoạn late trong flow, mọi thay đổi đều có nguy cơ "domino effect" — fix một violation có thể introduce violation khác. Incremental approach giữ tác động trong phạm vi kiểm soát được.

---

## Phần 5: Power And Area Recovery

### 5.1 Cơ Hội Recovery Xuất Hiện Từ Đâu?

Trong suốt quá trình optimization từ Synthesis đến CTS đến post-route:

- Cells được **upsize** (aggressive) để đảm bảo timing closure
- Nhiều **buffer được insert** để fix timing và SI
- **LVT cell** (Low-Vt, nhanh nhưng leaky) được dùng để hit timing target

Kết quả: Nhiều cell trong design có **positive slack dư thừa** (timing margin lớn hơn cần thiết). Đây là "lãng phí" — những cell này đang tiêu thêm power và chiếm area không cần thiết.

Post-route recovery khai thác positive slack này để improve PPA (Power, Performance, Area) trong khi vẫn đảm bảo timing.

### 5.2 Power Recovery Techniques

**Cell Downsizing:**

Thay cell có drive strength lớn bằng cell có drive strength nhỏ hơn tương đương cùng function, trên các path có đủ positive slack.

Tác động:
- Drive strength nhỏ hơn → transistor narrow hơn → **leakage power giảm** (tỷ lệ với W)
- Cell nhỏ hơn → physical footprint nhỏ hơn → area giảm (dual benefit)
- Cell nhỏ hơn → delay tăng → positive slack giảm (phải đủ slack để downsize)

**VT Swapping (Threshold Voltage Swap):**

Chuyển cell từ LVT (Low-Vt) sang HVT (High-Vt) trên non-critical path có đủ slack.

Nguyên lý transistor: Threshold voltage $V_t$ quyết định điểm bắt đầu transistor dẫn điện. Mối quan hệ giữa $V_t$ và leakage:

$$I_{leakage} \propto e^{-V_t / (nkT/q)}$$

Tức là $V_t$ càng cao → leakage giảm theo hàm mũ. Ngược lại, HVT cell chậm hơn LVT cell (cần thêm $V_t$ để "overcome"):

$$\text{Delay} \propto \frac{1}{\mu C_{ox} \frac{W}{L}(V_{GS} - V_t)^2}$$

Nên HVT chỉ áp dụng trên path có đủ slack để absorb thêm delay.

Trong thực tế foundry thường cung cấp ba loại Vt:
- **HVT** (High Vt): Chậm nhất, leakage thấp nhất
- **SVT/RVT** (Standard/Regular Vt): Cân bằng
- **LVT** (Low Vt): Nhanh nhất, leakage cao nhất
- Một số foundry còn có **ULVT** (Ultra Low Vt)

**Loại bỏ redundant buffer và inverter:**

Buffer pair (buffer inserted sau đó một buffer khác compensates) và inverter pair không cần thiết → xóa cả cặp để giảm cell count, power, và area.

### 5.3 Area Recovery Techniques

- **Cell downsizing** (như đã mô tả) giảm physical footprint trực tiếp
- **Remove redundant cells**: Buffer/inverter được insert trong earlier optimization stages (Synthesis, CTS) để fix timing có thể trở thành redundant sau post-route fix
- Cả hai giảm **cell utilization** của core — tạo thêm không gian cho future ECO nếu cần

### 5.4 Ràng Buộc Bắt Buộc

Mọi thay đổi trong Power/Area Recovery phải thỏa mãn **đồng thời**:

- Không vi phạm setup timing (positive slack không được xuống dưới 0)
- Không vi phạm hold timing
- Không tạo DRC violation mới (cell nhỏ hơn có thể cần legalization)
- Sau khi recovery: **bắt buộc re-run STA và DRC** để verify

---

## Phần 6: Chip Finishing Và DFM (Design For Manufacturability)

### 6.1 Mục Đích Của Chip Finishing

Chip Finishing là tập hợp các bước **hoàn thiện layout vật lý** sau khi timing và power đã được optimize. Mục tiêu không phải là timing hay function mà là:

- **Manufacturability**: Đảm bảo layout có thể được fabricate reliably
- **Reliability**: Giảm nguy cơ failure trong quá trình hoạt động và theo thời gian
- **Yield**: Tăng tỷ lệ chip pass sau fabrication

Thứ tự thực hiện trong chip finishing:

```
Filler cell insertion (non-cap) và Decap insertion
       ↓
Redundant via insertion
       ↓
Crosstalk fixing (còn lại)
       ↓
Wire spreading và Wire widening
       ↓
Metal filling
```

### 6.2 Filler Cell và Decap Cell

#### 6.2.1 Filler Cell

**Vấn đề cần giải quyết:**

Sau placement và routing, luôn tồn tại các **empty site** (vị trí trống) giữa standard cell trong cell row. Nếu để trống, hai vấn đề xảy ra:

1. **N-well và implant layer bị đứt đoạn**: Các lớp well và implant trong CMOS process phải liên tục xuyên suốt cell row. Đứt đoạn → DRC violation (well-related rule) → không thể fabricate.

2. **VDD/VSS power rail bị đứt**: Standard cell sử dụng power rail ngang chạy qua toàn bộ row. Nếu có khoảng trống, rail bị ngắt → cell trong row không nhận được nguồn → functional failure.

**Giải pháp:**

Filler cell là **physical-only cell** (không có chức năng logic, không kết nối signal net) được insert vào empty site. Chúng duy trì:
- Continuous N-well layer
- Continuous implant layer
- Continuous VDD/VSS power rail

**Cấu trúc thư viện filler:**

Foundry cung cấp filler cell với nhiều width: FILL1, FILL2, FILL4, FILL8, FILL16, FILL32, FILL64. Tool sẽ tự động chọn combination tối ưu (ví dụ: FILL16 + FILL4 + FILL1 = 21 site width) để lấp đầy hoàn toàn mọi khoảng trống có kích thước bất kỳ.

#### 6.2.2 Decap Cell (Decoupling Capacitor)

**Vấn đề cần giải quyết — Dynamic IR Drop:**

Khi nhiều cell **chuyển trạng thái đồng thời** (simultaneous switching), dòng điện $I_{switch}$ đột ngột tăng vọt. Dòng này phải được cung cấp từ nguồn điện qua power mesh. Tuy nhiên, power mesh có điện trở $R_{mesh}$ và inductance $L_{mesh}$:

$$V_{drop,dynamic} = I_{switch} \cdot R_{mesh} + L_{mesh} \cdot \frac{dI}{dt}$$

Điện áp tại cell giảm xuống tức thời → $V_{DD,local} < V_{DD,nominal}$ → cell chậm hơn (delay tăng theo $\frac{1}{V_{DD}-V_t}$) → timing violation.

**Nguyên lý decap:**

Decap cell có **tụ điện tích hợp** nối giữa VDD và VSS. Tụ điện này lưu trữ điện tích:

$$Q_{stored} = C_{decap} \cdot V_{DD}$$

Khi $V_{DD}$ sụt giảm cục bộ, tụ điện **xả ngay lập tức** bù vào trước khi dòng từ nguồn chính kịp đến (vì nguồn chính phải đi qua toàn bộ power mesh với delay tỷ lệ với $\sqrt{LC}$). Decap hoạt động như "local battery".

**Trade-off quan trọng:**

Decap cell tăng **leakage power** vì tụ điện trong MOS luôn có dòng rò qua lớp oxide mỏng. Đây là trade-off giữa dynamic IR drop improvement và static power penalty.

**Vị trí đặt decap:**

- Gần các cell có **switching activity cao** (high toggle rate)
- Tại các **IR drop hot spot** được xác định từ power analysis
- Có thể **pre-place trước placement** cho critical block, hoặc insert sau placement

**Thứ tự insert trong PnR tool:**

Tool ưu tiên insert **decap cell trước** để maximize on-chip decoupling capacitance, sau đó mới fill phần còn lại bằng non-cap filler cell. Lý do: Tối đa hóa capacitance trước, sau đó mới đảm bảo N-well continuity.

### 6.3 Redundant Via Insertion

#### 6.3.1 Tại Sao Single Via Là Rủi Ro?

Via là điểm kết nối giữa hai metal layer liền kề — đồng thời là **điểm dễ fail nhất** trong interconnect stack.

**Nguyên nhân 1 — Manufacturing defect:**

Trong quá trình lithography, mỗi mask layer có thể bị **misalign** (shift) vài nm so với layer khác (overlay error). Via phải có đủ enclosure metal để vẫn kết nối đúng khi có overlay error. Tuy nhiên, nếu overlay error lớn hơn enclosure → via không tiếp xúc đủ với metal → điện trở tăng đột biến hoặc open circuit.

Thêm vào đó, **random defect** (hạt bụi, particle contamination trong fabrication) có thể block via cut.

**Nguyên nhân 2 — Electromigration (EM):**

Dòng điện DC liên tục qua via tạo ra **lực điện tử** tác động lên nguyên tử kim loại trong hướng electron flow (electron wind force). Dần dần, nguyên tử bị dịch chuyển tạo ra **void** (chỗ rỗng) ở một đầu via và **hillock** (chỗ phình) ở đầu kia. Via hỏng dần theo thời gian (reliability degradation sau năm đến mười năm hoạt động). Đây là **long-term failure mechanism** — chip pass test ban đầu nhưng fail trong field.

**Nguyên nhân 3 — Thermal stress:**

Sự khác biệt giữa hệ số giãn nở nhiệt (CTE) của kim loại via và dielectric xung quanh → stress tích lũy qua mỗi chu kỳ nhiệt → metal voiding theo thời gian.

#### 6.3.2 Giải Pháp: Redundant Via

Thay single via bằng **hai hoặc nhiều via song song** tại cùng vị trí kết nối (multi-cut via). Lợi ích:

**Reliability**: Nếu một cut fail, net vẫn kết nối qua các cut còn lại.

**Resistance giảm (parallel connection):**

$$R_{via,total} = \frac{R_{via,single}}{N_{cuts}}$$

Điện trở tổng giảm theo số lượng cut song song → IR drop giảm → timing cải thiện nhẹ.

**EM margin tăng:**

Dòng điện được chia đều qua $N_{cuts}$ cut:

$$J_{each\_cut} = \frac{I_{total}}{N_{cuts}}$$

Current density mỗi cut giảm → EM risk giảm.

#### 6.3.3 Hai Thời Điểm Insert Redundant Via

**Trong routing (`route_design`):**

Router lên kế hoạch wire path với redundant via từ đầu. Ưu điểm: tỷ lệ insertion cao hơn vì router có nhiều routing resource. Nhược điểm: runtime dài hơn, có thể tăng congestion vì multi-cut via chiếm diện tích lớn hơn single-cut via.

**Sau timing closure (`route_opt`):**

Chỉ insert via tại vị trí có **đủ không gian** mà không cần modify existing wire → không tạo DRC mới, không thay đổi RC (không ảnh hưởng timing đã close).

**Via liability metric:**

Tool ưu tiên net có **via liability cao** — tức là net có nhiều single-cut via nhất hoặc via trên current-critical path. Tập trung resource vào những via có nguy cơ fail cao nhất.

**Lưu ý advanced node:**

Ở sub-14nm, foundry thường **bắt buộc** multi-cut via trên một số layer nhất định (double/triple patterning requirement). Đây không còn là optimization tùy chọn mà là DRC requirement.

### 6.4 Crosstalk Fixing (Post-Route)

#### 6.4.1 Bối Cảnh

Như L11 Section 15 đã trình bày, crosstalk mitigation đã được áp dụng **trong routing** thông qua SI-driven routing và NDR rules. Tuy nhiên, sau khi RC thực tế được extract, post-route noise analysis có thể phát hiện thêm violation.

Các violation này tồn tại vì:
- SI-driven routing chỉ dùng estimated coupling → có thể under-estimate actual coupling
- Congestion forced wire vào proximity chặt chịu hơn dự kiến
- Local density cao tạo ra coupling không được model chính xác ở global routing level

Liên hệ L11 Section 14: Coupling capacitance $C_m$ giữa aggressor và victim net là gốc rễ của mọi crosstalk effect. Sau extract RC, $C_m$ thực tế được biết chính xác — có thể còn lớn hơn ước tính.

#### 6.4.2 Kỹ Thuật Fix Manual ECO

**Wire Spacing (Tăng khoảng cách):**

Tăng khoảng cách giữa aggressor và victim vượt mức DRC minimum bằng NDR rule:

$$C_m \propto \frac{1}{d}$$

Trong đó $d$ là khoảng cách giữa hai wire. Spacing lớn hơn → $C_m$ giảm → coupling effect giảm.

Ứng dụng: Tạo NDR rule với spacing rule lớn hơn cho noise-sensitive net, áp dụng NDR cho victim net.

**Net Ordering (Sắp xếp thứ tự net):**

Sắp xếp lại vị trí relative của các net trong routing track sao cho victim không nằm cạnh worst aggressor. Giảm worst-case coupling **mà không tốn thêm routing resource**. Đây là low-cost fix đầu tiên cần thử.

**Shielding (Che chắn):**

Insert wire nối GND hoặc VDD chạy song song hai bên critical net. Wire shield (điện áp cố định, không switching) hấp thụ coupling từ aggressor trước khi đến victim.

Chi phí: Mỗi shield wire chiếm thêm routing track → tăng congestion và area. Cần cân nhắc trade-off với tác dụng SI.

**Re-buffer aggressor:**

Giảm drive strength của aggressor → switching current giảm → dV/dt giảm → coupling current giảm:

$$I_{coupling} = C_m \cdot \frac{dV_{aggressor}}{dt}$$

**Upsize victim driver:**

Driver mạnh hơn tăng noise immunity — victim có khả năng chống lại coupling noise tốt hơn vì low-impedance driver dập tắt coupling transient nhanh hơn.

**Reroute sang layer khác:**

Layer cao hơn thường có Pitch rộng hơn, mật độ wire thấp hơn → coupling capacitance thấp hơn. Layer với preferred direction khác → wire không chạy song song → ít coupling hơn.

#### 6.4.3 Mục Tiêu

Eliminate remaining **crosstalk-induced setup và hold violation** trước signoff. Đây là điều kiện cần trước khi pass STA signoff vì STA với SI enabled sẽ bắt các violation này.

### 6.5 Wire Spreading và Wire Widening

#### 6.5.1 Vấn Đề: Lithography-Induced Defects

Hai kỹ thuật này thuộc loại **DFM (Design For Manufacturability)** — không liên quan đến timing mà liên quan đến độ tin cậy của quá trình fabrication.

**Wire spreading giải quyết:** Sau detailed routing, nhiều wire nằm sát nhau ở khoảng cách minimum DRC spacing. Lithography thực tế không hoàn hảo — **pattern bridging** (hai wire dính vào nhau) có thể xảy ra khi khoảng cách quá nhỏ, đặc biệt ở lower metal layer (M1, M2) nơi Pitch nhỏ nhất.

**Wire widening giải quyết:** Wire có width tối thiểu dễ bị **open** (đứt) do **line-edge roughness (LER)** — hiện tượng cạnh wire không hoàn toàn thẳng sau lithography và etch, làm tiết diện thực tế nhỏ hơn thiết kế ở một số điểm.

Wire widening cũng cải thiện EM reliability vì:

$$J = \frac{I}{A} = \frac{I}{W \cdot T}$$

Width $W$ lớn hơn → diện tích tiết diện $A$ lớn hơn → current density $J$ thấp hơn → EM margin tăng.

#### 6.5.2 Wire Spreading Chi Tiết

**Nguyên lý:** Sau detailed routing, tự động tăng khoảng cách giữa wire liền kề **vượt minimum DRC** ở những chỗ có không gian trống. Không route lại wire — chỉ "đẩy" wire ra xa nhau hơn.

**Ưu tiên layer thấp:** Áp dụng đặc biệt quan trọng ở M1 và M2 nơi minimum spacing nhỏ nhất và bridge risk cao nhất.

**Tham số:** Spacing weight quyết định mức độ aggressive của tool khi tăng spacing (cao hơn = tích cực hơn).

#### 6.5.3 Wire Widening Chi Tiết

**Nguyên lý:** Tự động tăng width của wire **non-critical net** nơi có không gian. Chỉ áp dụng cho net không ảnh hưởng timing khi RC thay đổi.

**Tại sao không áp dụng cho critical net?**

Width tăng → $C_{wire}$ tăng nhưng $R_{wire}$ giảm:

$$R_{wire} = \rho \cdot \frac{L}{W \cdot T} \quad \text{(giảm khi W tăng)}$$

$$C_{wire} \propto W \cdot L \quad \text{(tăng khi W tăng)}$$

Net effect trên delay phụ thuộc vào tỷ lệ driver impedance và wire impedance. Trên critical path, bất kỳ thay đổi RC nào cũng có thể phá vỡ timing đã close → không áp dụng widening trên critical net.

**Ràng buộc bắt buộc:**

Sau spreading và widening, bắt buộc verify DRC clean (vì tăng width/giảm spacing có thể vi phạm rule với wire kề). Cũng phải verify timing không bị ảnh hưởng trên critical path.

### 6.6 Metal Filling

#### 6.6.1 Vấn Đề: CMP Non-Uniformity

Sau khi các metal layer được deposit và etch, foundry thực hiện **Chemical Mechanical Polishing (CMP)** để làm phẳng bề mặt trước khi deposit layer tiếp theo. CMP là quá trình mài bề mặt kết hợp hóa học và cơ học.

Vấn đề: CMP tốc độ không đồng đều theo metal density:

**Dishing:** Ở vùng metal density thấp (nhiều wire thưa), bề mặt dielectric bị mài quá nhiều → vùng đó thấp hơn xung quanh → wire resistance của net đi qua vùng đó sai lệch.

**Erosion:** Ở vùng metal density cao (nhiều wire dày), toàn bộ surface bị mài mòn không đều → wire resistance thực tế khác thiết kế.

**Dielectric loss:** Ở ranh giới giữa vùng dense và sparse, dielectric bị mài không đều → capacitance sai lệch.

Hậu quả: **RC thực tế sau CMP sai lệch** so với RC dự đoán từ nominal geometry → timing prediction sau extraction không chính xác → corner margin không đáng tin cậy.

Foundry enforce **minimum và maximum metal density rule per layer** trước tapeout — đây là DRC requirement cứng, không phải recommendation.

#### 6.6.2 Hai Loại Metal Fill

**Floating fill:**

Dummy metal shape insert vào vùng trống **không kết nối** với bất kỳ net nào. Mục đích đơn thuần là đáp ứng density requirement.

Nhược điểm: Floating conductor trong điện trường của wire gần đó tạo thêm **coupling capacitance** → tăng parasitics của neighboring net → potential timing impact.

**Grounded fill:**

Dummy metal shape được **kết nối xuống GND (VSS)**. Lợi ích kép:
- Đáp ứng density requirement (như floating fill)
- Hoạt động như **shield**: GND-connected fill hấp thụ coupling từ aggressor và không re-emit sang victim → reduce coupling capacitance và noise

Grounded fill được preferred over floating fill khi có thể.

#### 6.6.3 Timing-Aware Fill

Metal fill tăng parasitic capacitance của wire gần đó → delay tăng → có thể tạo timing violation trên critical path. Timing-aware fill giải quyết bằng:

**Fill keepout region:** Vùng cấm fill quanh critical net và clock net. Tool không insert fill trong vùng này.

**Insertion cost model:** Tool gán cost khác nhau tùy loại net:
- **PG net (Power/Ground)**: Zero cost — fill gần PG không ảnh hưởng timing
- **Signal net**: Moderate cost — fill gần signal tăng capacitance
- **Clock net**: High cost — fill gần clock ảnh hưởng skew và slew

Tool tối ưu hóa để đáp ứng density trong khi minimize tổng weighted cost.

**Selective removal:** Nếu timing analysis sau fill insert phát hiện violation mới, tool có thể selectively xóa fill gần các net bị ảnh hưởng.

#### 6.6.4 Metal Fill Shapes và Patterns

**Shape options:**

- **Square**: Hình vuông, dùng cho vùng có geometry constraints cụ thể
- **Rectangular**: Hình chữ nhật, linh hoạt hơn để match density target

**Pattern options:**

**Staggered pattern:** Fill trên layer liền kề được **offset** với nhau (không thẳng hàng theo chiều dọc). Lợi ích: Cross-coupling capacitance giữa fill trên layer khác nhau được phân tán — fill trên M3 không nằm thẳng bên trên fill trên M2 → coupling giảm. Đây là **default** cho lightly congested layer.

**Non-staggered pattern:** Fill thẳng hàng theo chiều dọc. Dùng khi foundry yêu cầu **uniform density distribution** nghiêm ngặt (một số layer có density uniformity constraint theo vùng rất nhỏ).

**Connectivity options:**

- Floating: Không kết nối — đơn giản, nhưng tạo floating conductor
- Connected to GND: Preferred — shield + density
- Connected to VDD: Cũng được dùng khi GND connection không thuận tiện
- Part of power structure: Fill được tích hợp vào power mesh

**Thứ tự thực hiện trong chip finishing:**

Metal fill là **bước physical cuối cùng** trước khi chạy signoff DRC và LVS. Lý do: Fill có thể tạo DRC violation mới (nếu fill quá gần existing wire) và phải được verified clean trước tapeout.

---

## Phần 7: Post-Route Signoff

### 7.1 Kiến Trúc Tổng Thể Của Signoff

Post-route signoff là tập hợp các verification bắt buộc trước tape-out. Sau khi PnR hoàn chỉnh, design phải vượt qua toàn bộ các check sau:

```
PnR Output (layout hoàn chỉnh)
              ↓
┌─────────────────────────────────────────────────────┐
│              PHYSICAL VERIFICATION                  │
│                                                     │
│   DRC          LVS         Antenna         ERC      │
│ (layout)   (connectivity) (gate oxide)  (electrical)│
└─────────────────────────────────────────────────────┘
         ↓ (parallel với physical verification)
┌─────────────────────────────────────────────────────┐
│         PARASITIC EXTRACTION & FORMAL               │
│                                                     │
│   Parasitic Extraction          LEC                 │
│   (RC từ layout geometry)  (netlist equivalence)    │
└─────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────┐
│              SIGNOFF ANALYSIS                       │
│                                                     │
│  Static Timing    Gate Simulation    Power EMIR     │
│  Analysis (STA)   (post-route)       Analysis       │
└─────────────────────────────────────────────────────┘
              ↓
           Tape-out
```

Bất kỳ check nào fail → feedback về PnR để fix → lặp lại cho đến khi all clean.

### 7.2 Physical Verification

#### 7.2.1 DRC — Design Rule Check

DRC verify rằng toàn bộ layout tuân thủ **geometric manufacturing rule** của foundry technology node. Foundry cung cấp DRC rule deck (Calibre runset, IC Validator deck) — đây là tài liệu chính thức từ fab, không do designer viết.

Các rule category chính:

- **Spacing rule**: Khoảng cách tối thiểu giữa hai shape cùng layer (ngăn bridge trong fabrication)
- **Width rule**: Width tối thiểu của wire và via (đảm bảo reliability)
- **Enclosure rule**: Metal phải bao quanh via cut đủ khoảng để chịu overlay error
- **Density rule**: Metal fill đã insert phải đáp ứng min/max density
- **Multi-patterning rule** (advanced node): Đảm bảo pattern có thể được in bằng multiple lithography steps

Liên kết L11 Section 24: DRC tồn tại vì fabrication không hoàn hảo — lithography, etch, deposition đều có variation. DRC rule là boundary đảm bảo design survive các variation này.

Tại advanced node (sub-14nm), DRC deck có hàng chục nghìn rule. Signoff DRC thường chạy bằng tool chuyên dụng (Mentor Calibre, Synopsys IC Validator), không phải in-tool DRC trong PnR.

#### 7.2.2 LVS — Layout vs. Schematic

LVS đảm bảo layout thực tế **khớp với netlist** về kết nối điện. Tool extract netlist từ GDS (dựa trên geometric connectivity của metal shapes), so sánh với post-route netlist từ PnR.

Các lỗi LVS phổ biến:

- **Short**: Hai net khác nhau bị nối nhau trong layout (do DRC violation hoặc routing error) → tạo kết nối không mong muốn
- **Open**: Một net bị đứt → hai endpoint không có đường dẫn điện → functional failure
- **Missing device**: Cell trong netlist không có tương ứng trong layout → placement bị mất hoặc GDS export error
- **Extra device**: Layout có cell không có trong netlist → ECO không được implement đúng

LVS và DRC thường chạy cùng nhau trong cùng tool environment. LVS clean là điều kiện **bắt buộc tuyệt đối** trước tape-out — foundry từ chối GDS có LVS error.

#### 7.2.3 Antenna Check

**Cơ chế vật lý:**

Trong quá trình plasma etch metal layer, **ion tích điện tích lũy** trên đoạn wire đang được etch (vì plasma là môi trường ion hóa). Nếu đoạn wire đó kết nối trực tiếp với gate oxide của transistor nhưng **chưa được kết nối với source/drain** (để discharge), điện tích tích lũy tạo ra điện áp:

$$V_{gate} = \frac{Q_{accumulated}}{C_{oxide}}$$

Nếu $V_{gate}$ đủ lớn → Fowler-Nordheim tunneling hoặc hot carrier injection phá hủy gate oxide → transistor degraded vĩnh viễn.

**Antenna ratio:**

$$\text{Antenna Ratio} = \frac{\text{Diện tích metal tích điện}}{\text{Diện tích gate oxide kết nối}}$$

Foundry quy định maximum antenna ratio cho từng layer. Layer cao hơn được etch sau → thời gian exposure trong plasma ngắn hơn → antenna ratio limit cao hơn.

**Fix methods:**

**Antenna diode**: Insert cell có diode nối giữa net và GND tại pin bị vi phạm. Diode cung cấp đường discharge cho điện tích trước khi đủ để phá oxide.

**Via jumper (Metal hopping)**: Route wire vi phạm lên layer cao hơn (etch sau → ít tích điện hơn). Wire tiếp tục trên layer cao, sau đó via xuống lại khi cần.

Liên kết L11 Section 17.3: Antenna diode insertion đã được enable trong routing (`route_detail_fix_antenna true` và `route_antenna_diode_insertion true`) — nhưng có thể vẫn còn sót violation sau routing. Signoff antenna check là final verification.

#### 7.2.4 ERC — Electrical Rule Check

ERC bắt các violation **điện học** mà DRC và LVS không phát hiện được.

**Common ERC checks:**

**Floating input**: Pin input của cell không kết nối với net nào. Trong CMOS, floating gate → transistor ở trạng thái không xác định → có thể draw đồng thời cả PMOS và NMOS → short mạch qua supply → continuous current flow → chip overheating.

**Multiple driver**: Hai output driver kết nối vào cùng net. Nếu một driver pull HIGH và driver kia pull LOW đồng thời → short circuit qua VDD-VSS → potentially destructive current.

**Missing power/ground connection**: Cell không có VDD hoặc VSS connection → không hoạt động, có thể draw leakage bất thường.

**ESD (Electrostatic Discharge) check:**

Điện tích tĩnh từ human body hoặc machine có thể đạt hàng nghìn volt → discharge qua IO pin → đánh thủng gate oxide hoặc làm nóng chảy junction.

ESD protection structure (thường là thyristor hoặc diode stack) đặt tại mỗi IO pad để:
- Clamp điện áp xuống mức an toàn khi có ESD event
- Shunt dòng ESD xuống GND trước khi vào mạch nội

ERC verify:
- Mỗi IO pin phải có qualified ESD protection cell trong pad ring
- Phải tồn tại discharge path liên tục từ mỗi IO đến power/ground clamp
- ESD cell phải tuân thủ ESD-specific spacing và placement rule trong PDK

ESD model chuẩn bao gồm HBM (Human Body Model ~2kV), MM (Machine Model ~200V), CDM (Charged Device Model ~500V) — mỗi model mô phỏng một nguồn discharge khác nhau với impedance và waveform khác nhau.

**Latchup check:**

Latchup là hiện tượng **parasitic PNPN thyristor** trong CMOS bulk process. Trong CMOS, cấu trúc NMOS và PMOS kề nhau tạo thành ký sinh NPN (N-well/P-sub/N+) và PNP (P+/N-well/P-sub) transistor. Nếu được trigger (bởi noise, overvoltage, radiation, hoặc substrate current):

- Hai transistor ký sinh **latch** vào trạng thái dẫn điện (positive feedback loop)
- Dòng VDD-VSS tăng đột biến (không giới hạn trừ bởi nguồn cấp)
- Chip bị destroy vĩnh viễn hoặc cần power cycle để reset

**Biện pháp:**

**Tap cell**: Cell đặc biệt cung cấp well contact (N-well tie to VDD, P-substrate tie to GND) để duy trì well potential ổn định, ngăn trigger latchup. Foundry quy định khoảng cách tối đa giữa tap cell — ERC verify rule này. Nếu tap cell quá thưa, latchup có thể xảy ra.

**Guard ring**: Vòng well contact bao quaround IO cell và sensitive block — cung cấp low-resistance shunt path để discharge substrate current trước khi trigger latchup. Well và substrate contact spacing phải đáp ứng PDK rule.

### 7.3 LEC — Logical Equivalence Check

#### 7.3.1 Vấn Đề Cần Giải Quyết

Trong suốt flow từ RTL đến post-route, nhiều transformation được thực hiện (synthesis optimization, CTS buffer insertion, ECO resize/swap/insert/delete). Mỗi step có nguy cơ **vô tình thay đổi logic function** của design.

LEC là **formal verification** — không simulation mà dùng mathematical proof (Boolean equivalence checking) để chứng minh hai netlist **logically identical**.

#### 7.3.2 Ba Thời Điểm Chạy LEC

**Sau Synthesis**: RTL vs. synthesized gate-level netlist → verify synthesis không làm sai logic trong quá trình optimize/restructure.

**Sau scan insertion**: Pre-scan netlist vs. post-scan netlist → verify DFT (Design For Test) scan chain insertion không ảnh hưởng functional logic ngoài scan functionality.

**Sau PnR**: Pre-route gate-level netlist vs. post-route gate-level netlist → verify ECO và optimization trong PnR không làm sai logic.

#### 7.3.3 Nguyên Lý Hoạt Động

LEC tool map **key points** (primary input, primary output, state element — Flip-flop D/Q pin) giữa hai netlist. Sau đó dùng **Boolean equivalence checking** (BDD — Binary Decision Diagram hoặc SAT solver) để prove:

$$f_{golden}(x_1, x_2, ..., x_n) \equiv f_{revised}(x_1, x_2, ..., x_n) \quad \forall (x_1, ..., x_n)$$

Nếu prove thành công → PASS (equivalent). Nếu SAT solver tìm được input vector phân biệt hai function → FAIL (non-equivalent) → ECO hoặc synthesis error.

#### 7.3.4 LEC Exception (Waiver)

Một số trường hợp LEC fail hợp lệ:
- **Flip-flop merge**: Hai Flip-flop được merge thành một (output vẫn equivalent nhưng key point mapping thay đổi)
- **Constant propagation**: Constant value được "baked in" — một số pin trở thành constant, LEC tool cần được inform
- **Scan insertion**: Pre-scan và post-scan netlist có logic khác nhau (intentionally) cho scan path

Trong những trường hợp này, engineer cung cấp **LEC exception** có justification. Không được phép waive lỗi LEC mà không có lý giải rõ ràng — đây là final safety net đảm bảo không có bug được introduce trong flow.

**Mục tiêu:** Guarantee no logic was inadvertently added, removed, or modified during implementation.

### 7.4 EM/IR Signoff

#### 7.4.1 Ba Loại Violation Và Cơ Chế Vật Lý

**Electromigration (EM):**

Dòng DC liên tục qua wire → electron wind force đẩy nguyên tử kim loại → void ở anode side, hillock ở cathode side → wire resistance tăng dần → cuối cùng open circuit.

Foundry định nghĩa **maximum current density** $J_{max}$ cho mỗi metal layer và via (đơn vị mA/μm²). Đây là giới hạn dưới điều kiện nhiệt độ và expected lifetime:

$$J = \frac{I}{W \cdot T} \leq J_{max}$$

EM violation không gây fail ngay mà là **long-term reliability degradation** — chip pass initial test nhưng có thể fail trong field sau vài năm.

**Static IR Drop:**

Voltage tại một cell = $V_{DD,supply}$ trừ voltage drop dọc power mesh:

$$V_{cell} = V_{DD} - I_{avg} \cdot R_{mesh}$$

Drop phụ thuộc vào dòng tiêu thụ trung bình $I_{avg}$ và điện trở của power mesh $R_{mesh}$ (từ nguồn đến cell). IR drop lớn → $V_{cell} < V_{DD,nominal}$ → cell delay tăng:

$$\text{Delay} \propto \frac{1}{(V_{DD} - V_t)^2}$$ 

(theo mô hình CMOS đơn giản, gần đúng) → nếu $V_{cell}$ thấp hơn nominal đủ nhiều, timing violation xảy ra.

**Dynamic IR Drop:**

Peak voltage drop khi nhiều cell switching đồng thời. Dòng đột biến $I_{switch}$ tạo voltage drop:

$$V_{drop,dynamic} = I_{switch} \cdot R_{mesh} + L_{mesh} \cdot \frac{dI_{switch}}{dt}$$

Thành phần inductive $L \cdot dI/dt$ thường dominant ở tốc độ cao. Dynamic IR drop **lớn hơn đáng kể** so với static IR drop và rất localized về thời gian (chỉ tồn tại trong vài clock cycle).

#### 7.4.2 Input Cần Thiết

EM/IR analysis không thể chạy chỉ từ netlist. Cần ba loại input:

- **Post-route extracted parasitics**: RC của toàn bộ power mesh — điều kiện tiên quyết
- **Switching activity**: VCD (Value Change Dump) từ gate simulation hoặc SAIF (Switching Activity Interchange Format) từ estimated toggle rate. Đây là input quan trọng nhất cho dynamic IR — không có switching activity → không tính được peak demand
- **Power intent**: UPF (Unified Power Format) hoặc CPF (Common Power Format) nếu design có multiple voltage domain (DVFS, power gating)

#### 7.4.3 Fix Violations

| Violation | Fix |
|-----------|-----|
| EM trên wire | Widen wire → tăng cross-section → giảm $J$ |
| EM trên via | Thêm via song song (redundant via) → chia dòng qua nhiều via |
| Dynamic IR drop hot spot | Insert decap cell gần hot spot → tăng local charge reservoir |
| Static IR drop | Widen power strap hoặc thêm power strap trong vùng có IR cao |
| Tổng quát | Strengthen power mesh, add more VDD/VSS connection |

**Mục tiêu:** Meet foundry EM limits và keep IR drop trong acceptable margins. Nếu fail → feedback về PnR để fix → re-run extraction → re-run EM/IR analysis.

### 7.5 STA Signoff

#### 7.5.1 Sự Khác Biệt Giữa In-Tool Timing Và STA Signoff

Trong suốt PnR, timing được check bằng **in-tool STA** (embedded trong Innovus, ICC2). Đây không phải signoff-quality vì:

- In-tool RC extraction: accuracy thấp hơn (dùng simplified model)
- In-tool STA: có thể dùng simplified delay model

**STA Signoff** dùng dedicated tool (Cadence Tempus, Synopsys PrimeTime) với:

- RC từ **dedicated extraction tool** (Cadence Quantus, Synopsys StarRC) — full-field parasitic solver, accuracy cao nhất
- **Signoff corners**: Foundry định nghĩa các tổ hợp Process-Voltage-Temperature (PVT) phải được verified. Ví dụ:
  - SS (Slow-Slow) / 0.72V / 125°C: Worst-case setup corner
  - FF (Fast-Fast) / 0.88V / -40°C: Worst-case hold corner
  - TT (Typical-Typical) / 0.8V / 25°C: Nominal
  - Thường 5-10 corners tùy foundry và design requirement
- **OCV (On-Chip Variation) derating**: Bù đắp cho sự biến đổi trong cùng một die:

$$\text{Cell delay with OCV} = \text{Nominal delay} \times (1 \pm \text{OCV derate factor})$$

Launch path được derated pessimistically (delay lớn hơn), capture path được derated optimistically (delay nhỏ hơn). Điều này tạo ra "net pessimism" — setup slack sẽ thấp hơn so với nominal.

**CPPR (Clock Path Pessimism Removal):** Khi OCV derating được áp dụng, common clock path (đoạn clock path chung giữa launch và capture) bị derate hai lần (một lần cho launch, một lần cho capture). CPPR loại bỏ pessimism này bằng cách cộng lại phần derating bị double-counted:

$$\text{Slack}_{CPPR} = \text{Slack}_{raw} + \text{CPPR credit}$$

#### 7.5.2 Điều Kiện Pass STA Signoff

- **Zero negative slack** trên tất cả corners và modes (setup và hold)
- Hoặc: Waiver có justification đầy đủ (path không reachable về functional, path được disable bởi scan mode, etc.)
- Waiver phải được approve bởi design lead và documented trong signoff report

#### 7.5.3 SDF Generation

Sau khi STA signoff clean, tool xuất **SDF (Standard Delay Format)** file. SDF chứa **annotated delay** cho từng cell và từng interconnect, được tổ chức theo từng corner.

**Nội dung SDF:**
- Cell instance delay (per corner)
- Interconnect delay (net wire delay per corner)
- Annotated for every cell and net in the design

**Mục đích:** Back-annotate delays vào **gate-level simulation** để simulation phản ánh đúng timing thực tế sau fabrication. SDF là cầu nối giữa static timing analysis và dynamic simulation.

SDF được generate per corner để capture:
- Worst-case delay (slow corner) → dùng cho setup verification trong simulation
- Best-case delay (fast corner) → dùng cho hold verification trong simulation

### 7.6 Gate Simulation Signoff (Post-Route Gate Simulation)

#### 7.6.1 Tại Sao STA Không Đủ?

STA là **static analysis** — phân tích timing trên từng path riêng lẻ theo mô hình tuyến tính:

- Không mô phỏng **dynamic behavior** của toàn bộ design đồng thời
- Không phát hiện **glitch**: tín hiệu chuyển trạng thái nhiều lần trong một clock cycle do race condition
- Không phát hiện **reset sequencing issue**: Thứ tự release reset giữa các domain không đúng
- Không phát hiện **X-propagation**: Trạng thái unknown lan rộng qua logic và mask violation thực sự (STA model unknown như 0 hoặc 1, không phải X)
- Không phát hiện **setup violation do multi-cycle path** được model không đúng trong constraint

#### 7.6.2 Gate-Level Simulation Với SDF

**Gate-level simulation** chạy functional simulation trên **post-route gate-level netlist** với delays được back-annotate từ SDF file.

**Flow:**

```
Post-route Netlist (gate-level)
SDF Files (per corner)          → Gate Simulation → Verify functionality
Test Patterns (vectors)
```

**Lưu ý quan trọng:** Gate-level simulation **KHÔNG phải full functional verification** — không đủ vector để cover tất cả state space của design. Chỉ check dynamic timing effects trên **selected test patterns** đại diện:

- Reset và initialization sequence
- Critical functional scenario
- Stress test (maximum switching activity)

**Mục tiêu:** Confirm timing clean và không có dynamic timing anomaly (glitch, X-propagation, reset issue) dưới điều kiện timing thực tế trước tape-out.

---

## Phần 8: Tổng Hợp — Signoff Checklist Trước Tape-out

Tất cả check sau phải **clean** (hoặc có waiver có justification) trước khi tape-out được approved:

| Check | Tool Category | Verify | Fail → Action |
|-------|--------------|--------|---------------|
| DRC | Physical Verification (Calibre/ICV) | Layout tuân thủ foundry geometric rule | Fix trong PnR hoặc manual ECO |
| LVS | Physical Verification | Layout connectivity khớp netlist | Fix connectivity error trong PnR |
| Antenna | Physical Verification | Không có gate oxide damage risk | Insert antenna diode hoặc re-route |
| ERC | Physical Verification | Không có electrical violation (floating, multi-drive, ESD, latchup) | Fix cell placement, tap cell spacing |
| LEC | Formal Verification (Conformal/Formality) | Post-route netlist logically equivalent với pre-route | Debug và fix ECO error |
| Parasitic Extraction | Extraction (Quantus/StarRC) | RC accuracy đủ cho signoff | Re-run extraction sau fix |
| STA Signoff | Timing (Tempus/PrimeTime) | Timing clean tất cả corners và modes | ECO fix, re-run STA |
| EM/IR | Power (Voltus/RedHawk) | EM và IR trong foundry limit | Widen wire, add via, add decap, strengthen mesh |
| Gate Simulation | Dynamic Simulation (VCS/Xcelium) | Không có dynamic timing anomaly | Debug và fix functional issue |

---

## Phần 9: Liên Kết Toàn Diện Giữa L11 và L12

### 9.1 Chuỗi Nhân Quả Xuyên Suốt Routing Flow

```
L11: Global Routing lập kế hoạch (GCELL, routing guide)
          ↓
L11: Detailed Routing tạo actual wire và via (exact RC unknown)
          ↓
[RC Extraction: estimated → actual]
          ↓
L12: Post-Route Optimization sử dụng actual RC
     ├── Timing Closure (fix violations với RC thực)
     ├── Power/Area Recovery (optimize với RC thực)
     └── Chip Finishing (DFM trên actual layout)
          ↓
L12: Signoff (verify tất cả với actual RC và layout)
          ↓
Tape-out
```

### 9.2 Khái Niệm L11 Được Mở Rộng Trong L12

| Khái niệm L11 | Mở rộng trong L12 |
|---------------|-------------------|
| Crosstalk (L11 §14): Coupling capacitance $C_m$ giữa wire | L12: Crosstalk fixing post-route — manual ECO để eliminate remaining violation sau SI-driven routing |
| NDR (L11 §10.3): Wire rộng hơn/spacing lớn hơn cho critical net | L12: Wire spreading/widening cho non-critical net (DFM), NDR tiếp tục trong crosstalk fix |
| Via types và redundant via (L11 §11): Multi-cut via cho reliability | L12: Redundant via insertion là chip finishing step riêng biệt với metric via liability |
| Post-route optimization (L11 §20): Concept tổng quan | L12: Chi tiết đầy đủ: ECO flow, selective re-buffering, useful skew, power recovery, VT swap |
| RC extraction (L11 §26): extract_rc sau routing | L12: RC là input bắt buộc cho EM/IR signoff và STA signoff |
| Timing path groups (L11 §22): in2reg, reg2reg, etc. | L12: Full clock path expansion (-path_type full_clock) để debug timing closure |
| DRC (L11 §24): Geometric rule | L12: DRC signoff với signoff-quality tool (Calibre) — phân biệt với in-tool DRC |
| Antenna effect (L11 §17.3): Fix trong routing | L12: Antenna check là separate signoff step, có thể vẫn vi phạm sau routing |

### 9.3 Tổng Quan PPA Impact Của Mỗi Giai Đoạn

| Giai đoạn | Power | Performance (Timing) | Area |
|-----------|-------|---------------------|------|
| Post-Route Timing Closure | Tăng (upsize, insert buffer) | Tốt hơn (fix violations) | Tăng |
| Power/Area Recovery | Giảm (downsize, HVT swap) | Neutral (trong margin) | Giảm |
| Filler/Decap insertion | Tăng (decap leakage) | Better (dynamic IR drop) | Không đổi (empty site) |
| Redundant via | Không đổi | Cải thiện nhẹ (R giảm) | Tăng nhẹ |
| Wire spreading/widening | Cải thiện (EM) | Không đổi (non-critical only) | Không đổi |
| Metal fill | Tăng nhẹ (floating cap) | Có thể ảnh hưởng (timing-aware fill) | Không đổi |

---

## Phần 10: Các Nguyên Lý Được Suy Ra Từ Tool Commands

Phần này extract lý thuyết từ các command trong slide — không mô tả syntax mà phân tích ý nghĩa vật lý/kỹ thuật.

### 10.1 Routing Setup Strategy (Từ route_settings.tcl, p3)

**Bottom routing layer là M2, không phải M1:** Trong standard cell, M1 được dùng nội bộ bởi cell (pin connection, power rail bên trong cell). Tool không được route tín hiệu trên M1 ngoài phạm vi cell → bottom routing layer phải là M2. Đây là quy ước chuẩn trong standard cell flow.

**Timing-driven và SI-driven được enable đồng thời:** Router phải xem xét cả timing cost và SI cost khi chọn path. Đây là multi-objective optimization — có trade-off giữa timing (path ngắn, ít via) và SI (spacing lớn, tránh parallel run dài).

**Antenna fixing được enable trong routing:** Router chịu trách nhiệm insert antenna diode hoặc wire jogging ngay trong routing — không defer đến post-route step. Tuy nhiên, signoff antenna check vẫn cần thiết để catch remaining violation.

**Clock net fixing được enable:** Router được phép modify wire của clock net nếu cần để fix DRC hoặc timing. Điều này khác biệt với placement stage — sau CTS, clock net thường được "frozen" nhưng trong routing, một số adjustment vẫn được phép.

**route_special trước route_design:** Power/ground net phải được route đầy đủ trước khi signal routing bắt đầu. Lý do: Power distribution cần được đảm bảo trước, và power net rất wide (NDR) — phải biết chúng ở đâu trước để signal routing tránh.

### 10.2 Multi-Cut Via Strategy (Từ additional_route_settings.tcl, p3)

**Reserve space cho multi-cut via từ đầu:** Nếu không reserve, router sẽ route wire quá gần nhau — không còn không gian cho multi-cut via sau đó. Reserve space là proactive DFM decision.

**Via weight và concurrent minimization:** Router có thể trade off via count vs. wire length — ít via hơn nhưng wire dài hơn, hoặc ngược lại. Via weight parameter điều chỉnh trade-off này. Minimize via count giảm RC và cải thiện timing/reliability.

**Post-route via swap:** Sau routing hoàn thành, tool có thể swap single-cut via thành multi-cut via nơi có không gian — incremental improvement mà không thay đổi wire topology.

**Via pillar effort:** Via pillar là stack of multiple vias trên nhiều layer tại cùng (x,y) location — cực kỳ hữu ích để kết nối từ layer thấp lên layer cao trong vùng congested. Effort level quyết định resource tool dùng để optimize via pillar placement.

### 10.3 Litho-Driven Routing (Từ additional_route_settings.tcl, p3)

**Litho-driven routing:** Router nhận biết các pattern khó in (90-degree notch, narrow space trước wide space, v.v.) và tránh tạo ra chúng. Đây là DFM consideration được integrate vào routing stage — fix lithography issue **trước** khi chúng trở thành DRC hay yield issue sau fabrication.

**Litho repair post-route:** Sau routing, tool scan toàn bộ layout và sửa các pattern có litho risk — tương tự wire spreading/widening nhưng focused on printability thay vì EM.

**Trim metal:** Loại bỏ các đoạn metal ngắn không cần thiết (trim) sau routing. Những đoạn metal này có thể tạo antenna risk hoặc unintended connection.

### 10.4 DFM Settings (Từ dfm_settings.tcl, p31)

**Filler cell hierarchy:** Filler được cung cấp theo kích thước 2^n (FILL1, FILL2, FILL4, FILL8, FILL16, FILL32, FILL64) để tool có thể combine chính xác bất kỳ kích thước trống nào bằng binary decomposition.

**Decap cell có prefix riêng (DCAP):** Phân biệt clearly với non-cap filler (FILL prefix). Tool insert DCAP trước để maximize capacitance, sau đó FILL cho phần còn lại.

**Antenna diode cell có tên cụ thể (ANTENNACELL):** Mỗi foundry cung cấp antenna diode cell chuyên dụng với specific DRC compliance. Không thể dùng generic diode.

**metal_fill_timing_aware:** Flag này enable toàn bộ timing-aware fill mechanism — cost model, keepout region, selective removal. Phải enable trước khi chạy add_metal_fill để tránh insert fill vô điều kiện gây timing regression.

### 10.5 ECO Command Structure (Từ setup_eco.tcl, p31)

Từ cấu trúc ECO command, có thể suy ra **granularity của ECO change**:

- Từng cell có thể được add/resize/swap/delete với instance name và location cụ thể
- Từng net có thể được add
- Từng pin connection có thể được establish hoặc break

Granularity này cho phép thay đổi **cực kỳ localized** — chỉ touch đúng cell và net cần fix, không ảnh hưởng phần còn lại. Đây là lý do ECO có thể được thực hiện late in flow mà không destabilize design.

**eco_mode trong placement và routing:** Khi placement hoặc routing chạy với eco_mode = true, tool biết đây là incremental change — không re-optimize toàn bộ mà chỉ legalize/route những gì bị thay đổi. Điều này đảm bảo "surgical" nature của ECO.

**reroute modified_nets_only:** Chỉ reroute các net có topology thay đổi do ECO — không touch bất kỳ net nào khác. Đây là key principle của ECO routing: **preserve as much existing routing as possible**.

---

*Tài liệu này được tổng hợp từ Lecture 12 – Routing (Part II), Tresemi, với liên kết đối chiếu từ Lecture 11 – Routing (Part I). Tất cả lý thuyết, công thức, và nguyên lý được trình bày trên là nội dung gốc từ hai lecture, không thêm nội dung ngoài phạm vi.*
