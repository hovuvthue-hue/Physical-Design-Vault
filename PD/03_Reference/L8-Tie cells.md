Trang 21, 22, 23 trong file L8-Placement_2

#### 1. Vấn đề cần giải quyết: Tại sao không được nối Gate trực tiếp vào VDD/VSS

Gate oxide nằm dưới poly gate là phần **mỏng nhất và nhạy cảm nhất** của transistor. Có hai nguy cơ phá hủy gate oxide:
- **Antenna effect trong fabrication**: trong quá trình etching kim loại, charge tích tụ trên metal wire dài → discharge qua gate oxide → oxide bị phá hủy vĩnh viễn.
- **Voltage surge trong operation**: kết nối trực tiếp Gate → VDD/VSS tạo đường dẫn cứng → transient voltage spike có thể vượt ngưỡng chịu đựng của oxide.

**Kết luận:** Direct connection của input gate Pin đến VDD hoặc VSS là **không được phép** trong thiết kế thực tế.
#### 2. Giải pháp: Tie Cells và nguyên lý hoạt động

Tie Cell là Standard Cell đặc biệt dùng để cung cấp logic constant '0' hoặc '1' một cách an toàn. Bên trong Tie Cell sử dụng **diode-connected transistor**:
```
Vg = Vd
→ Vgs = Vds
→ Vds > Vgs - Vt  (transistor luôn ở saturation)
```
Cấu hình diode-connected tạo ra **impedance hữu hạn** giữa nguồn điện và gate oxide — điện áp surge bị hấp thụ thay vì đi thẳng vào gate → oxide được bảo vệ.

Hai loại Tie Cell:

|Cell|Output|Thay thế|
|---|---|---|
|**Tie-high cell**|Logic '1' (kết nối lên VDD qua PMOS diode)|Tất cả `1'b1` trong Netlist|
|**Tie-low cell**|Logic '0' (kết nối xuống VSS qua NMOS diode)|Tất cả `1'b0` trong Netlist|
#### 3. Vị trí Tie Cell trong flow và trạng thái Netlist

**Tie Cell là Standard Cell nhưng KHÔNG có trong incoming Netlist** (Netlist từ Synthesis). Chúng được insert trong Physical Design flow, sau Placement.

**Netlist trước Tie Cell Insertion:**
verilog
```verilog
SDFFQX2 DFF (.CK(CP), .D(1'b0), .Q(Q), .SE(SE), .SI(SI));
```
**Netlist sau Tie Cell Insertion:**
verilog
```verilog
wire LTIELO_NET;
TIELO LTIELO (.Y(LTIELO_NET));
SDFFQX2 DFF (.CK(CP), .D(LTIELO_NET), .Q(Q), .SE(SE), .SI(SI));
```
Tất cả `1'b0` → được thay bằng output của Tie-low cell qua một Net trung gian. Tương tự với `1'b1` → Tie-high cell.
#### 4. Constraints cho Tie Cell Insertion

Hai constraint quan trọng cần set trước khi insert:
- **Max fanout**: một Tie Cell không được drive quá N gate inputs — nếu vượt quá, tool tự động insert thêm Tie Cell mới.
- **Max distance**: khoảng cách tối đa giữa Tie Cell và các Cell nó drive — đảm bảo wirelength không quá dài, tránh IR drop trên Net constant.

#### 5. Lệnh thực thi theo thứ tự
tcl
```tcl
# Set constraints
set_db add_tieoffs_max_fanout 10      # mỗi Tie Cell drive tối đa 10 inputs
set_db add_tieoffs_max_distance 20   # khoảng cách tối đa 20 microns
set_db add_tieoffs_cells {TIEHI TIELO}  # chỉ định tên Cell trong thư viện

# Insert Tie Cells
add_tieoffs

# Verify sau khi insert
check_place          # kiểm tra Placement hợp lệ sau khi thêm Cell mới
verify_connectivity  # kiểm tra tất cả Net được kết nối đúng
report_tieoffs       # report số lượng Tie Cell đã được insert
```

**Kết quả thực tế từ slide 23:**
```
Total Number of Tie Cells (TIEHI) placed: 16
Total Number of Tie Cells (TIELO) placed: 409
Re-routed 164 nets (sau TIEHI)
Re-routed 409 nets (sau TIELO)
```
Tool tự động re-route các Net bị ảnh hưởng sau khi insert Tie Cells.
#### 6. Chọn Tie Cell để visualize trong GUI

tcl
```tcl
gui_deselect -all
select_obj [get_db insts -if {.name == *LTIELO}]
```
Lệnh này select tất cả instance có tên chứa "LTIELO" để kiểm tra vị trí Placement trong layout.
#### Tóm tắt logic
```
Synthesis Netlist chứa 1'b0 và 1'b1
        ↓
Direct connection Gate → VDD/VSS nguy hiểm cho gate oxide
(antenna effect + voltage surge)
        ↓
set constraints (max_fanout, max_distance, cell names)
        ↓
add_tieoffs
        ↓
1'b0 → TIELO cell → output Net → gate input
1'b1 → TIEHI cell → output Net → gate input
        ↓
check_place + verify_connectivity + report_tieoffs
```