Trang 26, 27 trong file L8-Placement_2
### 1. Khái niệm cốt lõi: MBFF là gì

MBFF (Multi-Bit Flip-Flop) là một Flip-flop đơn lẻ chứa nhiều bit bên trong, chia sẻ chung clock buffer và clock tree connection. Tool thực hiện **banking** — gộp nhiều single-bit Flip-flop riêng lẻ lại thành một MBFF tương đương về mặt chức năng.

Ví dụ cụ thể từ slide:
```
8 × 1-bit FF  →  1 × 8-bit MBFF
4 × 2-bit FF  →  2 × 4-bit MBFF
2 × 1-bit FF  →  1 × 2-bit MBFF (minh họa trong hình)
```
### 2. Điều kiện bắt buộc để tool merge

Tool **chỉ merge** các Flip-flop khi:
- Chúng có **cùng Timing constraints** (same Setup/Hold requirements, same clock domain).
- Constraints được **copy nguyên vẹn** sang MBFF kết quả — không mất thông tin Timing.

Nếu hai Flip-flop thuộc hai clock domain khác nhau hoặc có SDC constraints khác nhau → tool không merge.
### 3. Bốn lợi ích của MBFF (theo thứ tự quan trọng)

**Lợi ích 1 — Area reduction:** MBFF chia sẻ transistor bên trong (shared clock buffer transistors, shared well contacts, optimized transistor-level layout) → diện tích tổng nhỏ hơn tổng diện tích các single-bit FF riêng lẻ.

**Lợi ích 2 — Power reduction:**
- Clock input của nhiều bit chỉ load **một điểm kết nối** thay vì N điểm → clock net capacitance giảm → dynamic power giảm.
- Area nhỏ hơn → leakage giảm.

**Lợi ích 3 — Clock tree net length reduction:** Thay vì clock tree phải branch đến N vị trí riêng lẻ → chỉ cần đến một vị trí MBFF → tổng clock tree wirelength giảm đáng kể.

**Lợi ích 4 — Better clock skew control / Timing improvement:** Các bit trong cùng một MBFF nhận clock **tại cùng một điểm vật lý** → Skew giữa các bit đó = 0 → CTS dễ balance hơn → Timing tốt hơn.

### 4. Vị trí trong flow

MBFF Optimization diễn ra trong **post-placement optimization stage**, sau Power/Area optimization, trước Tie Cell insertion. Được thực thi thông qua `opt_design -pre_cts` khi đã bật flag MBFF.

### 5. Lệnh thực thi theo thứ tự (Slide 27)

```tcl
# Bật MBFF optimization
set_db opt_multi_bit_flop_opt true

# Đặt prefix cho tên instance MBFF được tạo ra
set_db opt_multi_bit_flop_name_prefix MBIT_

# Chạy optimization (MBFF được thực hiện bên trong opt_design)
opt_design -pre_cts

# Report kết quả MBFF
report_multibit
```
### 6. Đọc report_multibit (Slide 27)

```
Current Design Sequential Cells Statistics
                    Latches    Flops    All Seq Cells
Single-Bit Count      0        11531        11531
Multi-Bit Count        0            0            0
Total Bit Count        0        11531        11531
  -Merged             0            0            0
  -Un-Merged          0        11531        11531
Multibit Conversion %           0.00         0.00
Total Clock Pin Cap             6695.89
```

**Phân tích kết quả này:** Trong ví dụ slide 27, `Multibit Conversion % = 0.00` vì log cho biết **"No Multi Bit Sequential Cells Present in the Library"** — thư viện không có MBFF cells → tool không thể merge dù đã bật flag. Đây là trường hợp thực tế quan trọng cần nhớ: **MBFF optimization chỉ hoạt động khi thư viện có sẵn MBFF cell types**.

Các metric cần theo dõi khi MBFF hoạt động thành công:

- **Multi-Bit Count**: số MBFF được tạo ra.
- **Merged**: số FF đã được banking vào MBFF.
- **Multibit Conversion %**: tỉ lệ % FF được convert → càng cao càng tốt.
- **Total Clock Pin Cap**: tổng clock capacitance → giảm sau MBFF = power savings.

### Tóm tắt logic

```
N × single-bit FF riêng lẻ → N clock tree connections riêng lẻ
                             → N vị trí Placement riêng lẻ
        ↓
set_db opt_multi_bit_flop_opt true
set_db opt_multi_bit_flop_name_prefix MBIT_
opt_design -pre_cts
        ↓
Điều kiện: same Timing constraints + thư viện có MBFF cells
        ↓
Banking: N × 1-bit FF → 1 × N-bit MBFF
        ↓
Area giảm (shared transistors)
Power giảm (ít clock net capacitance)
Clock tree ngắn hơn (ít branch points)
Skew giảm (cùng clock point)
        ↓
report_multibit → kiểm tra Conversion %
```