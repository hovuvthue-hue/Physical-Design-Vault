Trang 24, 25 trong file L8-Placement_2
## Spare Cells — Slide 24 & 25

### 1. Khái niệm cốt lõi: Spare Cells là gì và tại sao cần thiết

Spare Cells là các Standard Cell được insert vào layout **không phục vụ bất kỳ chức năng logic nào tại thời điểm Tape-out** — chúng được dự phòng cho **[[4.2. Standard Cell Libraries#^ECO-cell|ECO]] (Engineering Change Order)** trong tương lai. ECO là các thay đổi logic cần thiết sau khi design đã được sign-off hoặc thậm chí sau khi chip đã được fabricate (metal-only ECO).

**Vấn đề nếu không có Spare Cells:** Khi cần ECO, engineer phải tìm chỗ trống trong layout để thêm Cell mới — nếu không có chỗ trống gần vùng cần sửa → phải re-place, re-route toàn bộ vùng đó → tốn thời gian và chi phí cực lớn.

**Giải pháp:** Pre-insert Spare Cells được phân bố đều khắp design → khi ECO cần Cell mới, lấy Spare Cell gần nhất → chỉ cần thay đổi Metal layer connection → không cần re-Placement.

### 2. Đặc điểm của Spare Cells trong layout

- **Input:** được tie đến VDD/VSS thông qua [[Tie cells]] → Cell ở trạng thái known logic state, không switching → không tiêu thụ dynamic power.
- **Output:** để hở (unused/floating) — chỉ kết nối khi ECO cần dùng.
- **Thành phần:** bao gồm mix các Cell type: combinational cells (Inverter, Buffer, NAND, NOR, AND, OR) và Flip-flop — để ECO có đủ loại Cell cần dùng.
- **Phân bố:** rải đều khắp Core area với step cố định → đảm bảo bất kỳ vùng nào trong design cũng có Spare Cell gần đó.

### 3. Lệnh thực thi theo thứ tự (Slide 25)

**Bước 1 — Tạo spare module với danh sách Cell types:**
```tcl
create_spare_module \
    -module_name spare_1 \
    -cells {INVX1 INVX2 NAND2X1 NOR2X1 BUF_X1 DFFHQX1}
```
- `-module_name spare_1`: đặt tên cho spare module.
- `-cells {...}`: liệt kê các Cell type sẽ có trong mỗi spare module instance — cần đa dạng để phục vụ nhiều loại ECO khác nhau.

**Bước 2 — Place spare modules với step placement:**
```tcl
place_spare_modules \
    -module_name spare_1 \
    -step_x 50 \
    -offset_x 10 \
    -step_Y 50 \
    -offset_Y 10
```
- `-step_x 50 / -step_Y 50`: khoảng cách giữa các spare module instance theo chiều X và Y (đơn vị micron) — xác định mật độ phân bố.
- `-offset_x 10 / -offset_Y 10`: điểm bắt đầu đặt spare module đầu tiên tính từ origin của Core.

**Bước 3 — Legalize và verify:**
```tcl
legalize_placement   # đảm bảo Spare Cells không overlap với Cells khác
check_place          # verify toàn bộ Placement hợp lệ
```
### 4. Netlist được generate sau khi insert Spare Cells

Tool tự động tạo module con và kết nối inputs vào Tie Cells:
```verilog
module spare_1_10 (...);
    wire tie_hi_net0;
    
    ADDFX2  ADDFX2_spr_gate10  (.A(tie_hi_net0), .B(tie_hi_net0), ...);
    AND2X4  AND2X4_spr_gate9   (.A(tie_hi_net0), .B(tie_hi_net0));
    BUFX2   BUFX2_spr_gate7    (.A(tie_hi_net0));
    INVX2   INVX2_spr_gate5    (.A(tie_hi_net0));
    SDFFQX1 SDFFQX1_spr_gate1  (.CK(tie_hi_net0), .D(tie_hi_net0), ...);
    TIEHI   TIEHI_spr_gate0    (.Y(tie_hi_net0));   // Tie Cell cung cấp '1'
endmodule
```
Tất cả inputs được kết nối vào `tie_hi_net0` (output của TIEHI Tie Cell) → Cell ở known state, không float.
### 5. Visualize Spare Cells trong GUI

```tcl
# Select tất cả spare cells để kiểm tra phân bố
select_obj [get_db insts -if {.name == *spr_gate*}]
```
### 6. Trade-off khi dùng Spare Cells

|Lợi ích|Chi phí|
|---|---|
|ECO nhanh, không cần re-Placement|Tăng die area (chiếm chỗ trong Core)|
|Giảm thời gian và chi phí ECO|Tăng nhẹ leakage power (dù inputs được tied)|
|Không ảnh hưởng Timing hiện tại|Cần lên kế hoạch tỉ lệ % spare hợp lý|

Tỉ lệ Spare Cells thông thường trong công nghiệp: **2–5% tổng số Cell** trong design, tùy mức độ rủi ro ECO dự đoán.
### Tóm tắt logic

```
ECO sau Tape-out cần Cell mới gần vùng cần sửa
        ↓
Không có Spare → phải re-Place, re-Route → chi phí cao
        ↓
create_spare_module (định nghĩa Cell types)
        ↓
place_spare_modules (step_x, step_Y → phân bố đều)
        ↓
Inputs tied to VDD/VSS qua Tie Cells (no floating, no switching)
Outputs để hở
        ↓
legalize_placement → check_place
        ↓
ECO cần Cell → lấy Spare Cell gần nhất → chỉ thay đổi Metal → xong
```