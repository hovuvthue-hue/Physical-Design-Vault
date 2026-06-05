---
tags: [concept, pnr-flow, placement, timing]
group: PnR Flow
defined_in: Pre-CTS Optimization — group_path commands trước opt_design
used_by: [PreCTSOptimization, STA, Placement]
requires: [STA, Slack, GateLevelNetlist]
chain: Chain_PnR_Flow
---
# PathGrouping

## Definition
PathGrouping là việc phân loại timing paths thành các nhóm có tên để STA tool và optimization engine có thể áp dụng chiến lược phân tích và tối ưu khác nhau cho từng nhóm. Thay vì xử lý toàn bộ paths như một khối, việc phân nhóm cho phép engineer kiểm soát effort level per group — tập trung tài nguyên optimization vào các nhóm quan trọng nhất.

## Primary Path Groups

Bốn nhóm timing path chính, phân loại theo loại endpoint:

| Group | Startpoint | Endpoint | Đặc điểm |
|---|---|---|---|
| **input** (in2reg) | Primary Input Port | Register (FF data pin) | Phụ thuộc `set_input_delay` trong SDC |
| **reg2reg** | Register clock pin | Register data pin | Nhóm nội bộ quan trọng nhất về số lượng |
| **in2out** | Primary Input Port | Primary Output Port | Thường không có sequential element |
| **output** (reg2out) | Register clock pin | Primary Output Port | Phụ thuộc `set_output_delay` trong SDC |

## Additional Path Groups

Ba nhóm bổ sung cho các loại endpoint đặc biệt:

| Group | Startpoint | Endpoint |
|---|---|---|
| **in2icg** | Primary Input | ICG (Integrated Clock Gating) cell |
| **reg2icg** | Register | ICG cell |
| **reg2mem** | Register | Memory (black box) |
| **mem2reg** | Memory | Register |

## Effort Levels

Mỗi group được gán một effort level để điều chỉnh mức độ tối ưu:

| Group | Effort Level | Lý do |
|---|---|---|
| input | low | I/O paths thường được model bởi SDC constraints, ít complex |
| output | low | Tương tự input |
| in2out | low | Combinational path đơn giản, ít tác động nếu fail |
| **default** | **high** | Paths không thuộc group nào — cần xử lý thận trọng |
| **reg2reg** | **high** | Nhóm quan trọng nhất, nhiều violations tiềm năng |
| **reg2mem** | **high** | Memory paths critical |
| **mem2reg** | **high** | Memory paths critical |
| **reg2icg** | **high** | ICG paths ảnh hưởng đến power gating |
| in2icg | low | Input-to-ICG ít critical hơn reg2icg |

## Why Path Grouping Matters

Không có grouping, optimizer xử lý tất cả paths với cùng priority — dẫn đến lãng phí effort vào các paths ít quan trọng (in2out, input) trong khi có thể thiếu effort cho các paths critical (reg2reg). Grouping cũng giúp:
- Debug dễ hơn: timing report per group thay vì toàn chip
- Targeted optimization: upsize cells chỉ trên paths specific groups cần
- Report phân tách: WNS/TNS/FEP per group cho visibility rõ ràng

## Commands

```tcl
# Reset existing groups
reset_path_group -all
reset_path_group_options

# Define primary groups
group_path -name input   -from [all_inputs -no_clocks] -to [all_registers]
group_path -name output  -from [all_registers] -to [all_outputs]
group_path -name in2out  -from [all_inputs -no_clocks] -to [all_outputs]
group_path -name reg2reg -from [filter_collection [all_registers] "is_black_box != true"] \
                         -to   [filter_collection [all_registers] "is_black_box != true"]

# Additional groups
group_path -name reg2mem -from [filter_collection [all_registers] "is_black_box != true"] \
                         -to   [filter_collection [all_registers] "is_black_box == true"]
group_path -name mem2reg -from [filter_collection [all_registers] "is_black_box == true"] \
                         -to   [filter_collection [all_registers] "is_black_box != true"]
group_path -name reg2icg -from [filter_collection [all_registers] "is_black_box != true"] \
                         -to   [filter_collection [all_registers] "is_clock_gating_check == true"]
group_path -name in2icg  -from [all_inputs -no_clocks] \
                         -to   [filter_collection [all_registers] "is_clock_gating_check == true"]

# Set effort levels
set_path_group_options input   -effort_level low
set_path_group_options output  -effort_level low
set_path_group_options in2out  -effort_level low
set_path_group_options default -effort_level high
set_path_group_options reg2reg -effort_level high
set_path_group_options reg2mem -effort_level high
set_path_group_options mem2reg -effort_level high
set_path_group_options reg2icg -effort_level high
set_path_group_options in2icg  -effort_level low
```

## Position in Pre-CTS Flow
PathGrouping thường được thiết lập ngay sau khi activate Analysis Views và trước khi chạy `opt_design -pre_cts`. Các groups được source từ một file riêng để tái sử dụng:

```tcl
source ./group_paths.tcl   # chứa tất cả group_path commands
opt_design -pre_cts -setup
```

## Requires
- [[STA]]
- [[Slack]]
- [[GateLevelNetlist]]

## Used by
- [[PreCTSOptimization]] — path grouping là một bước setup quan trọng trước `opt_design -pre_cts`
- [[STA]] — timing report per group giúp identify violation concentration
- [[Placement]] — optimization engine dùng group definitions để phân bổ effort

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Position in flow: trước [[PreCTSOptimization]] → [[ClockTreeSynthesis]]
→ Related: [[STA]] · [[Slack]] · [[PreCTSOptimization]]
→ Cùng nhóm: [[PreCTSOptimization]] · [[DRVFixing]] · [[PowerOptimization]]