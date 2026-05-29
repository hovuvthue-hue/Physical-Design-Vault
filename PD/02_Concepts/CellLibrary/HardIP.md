---
tags: [concept, cell-library]
group: Cell Library
defined_in: IP vendor / Foundry — delivered as fixed layout (GDS + LEF + LIB + Verilog)
used_by: [Floorplanning, Placement, Routing, STA, Signoff]
requires: [LEF, LIB, GDS, CellAbstract]
chain: Chain_LEF_to_PnR
---

# HardIP

## Definition
Hard IP (Hard Intellectual Property) là một khối chức năng được thiết kế và tối ưu hóa sẵn với layout cố định (fixed layout) — PD engineer chỉ có thể integrate nó vào chip mà không thể thay đổi internal implementation. Ngược lại với Soft IP (delivered as synthesizable RTL, treated như regular RTL blocks), Hard IP được giao dưới dạng GDS + LEF + LIB + Verilog — tức là đã hoàn chỉnh ở physical level. Hard IPs thường là các blocks đòi hỏi custom analog/mixed-signal design hoặc memory architecture không thể đạt được chất lượng tương đương bằng automated digital flow.

## Computed from
Hard IP không được generated trong digital PD flow — nó là pre-designed artifact từ IP vendor hoặc Foundry internal team. Các loại Hard IP phổ biến:

**Memory Hard IPs**: SRAM, ROM, NVM (Flash, eFuse) — generated bởi Memory Compiler tool; PD engineer specify configuration (word count, bit width, mux ratio) và Memory Compiler output toàn bộ views.

**Analog/Mixed-Signal IPs**: PLL (Phase-Locked Loop), ADC, DAC, SERDES, PHY (USB, PCIe, DDR) — designed thủ công bởi analog engineers; layout được done in custom tools.

**Standard Interface PHY IPs**: USB PHY, PCIe PHY, Ethernet MAC/PHY — thường licensed từ IP vendors (Synopsys DesignWare, Cadence IP).

**Views bắt buộc phải có để integrate Hard IP vào PnR:**

| View | Format | Dùng cho |
|---|---|---|
| Physical Abstract | LEF | Placement và Routing (Floorplan, P&R) |
| Timing Model | LIB | STA — Setup/Hold analysis |
| Gate-level Netlist | Verilog | Functional simulation |
| Transistor Netlist | CDL/SPICE | LVS verification |
| Full Layout | GDS | DRC và Tape-out |

**Memory Compiler workflow**: PD engineer → specify config → Memory Compiler generates tất cả 5 views automatically → integrate vào PnR flow.

## Constrains
- **[[Floorplanning]]**: Hard IP footprint (từ LEF SIZE) là fixed và phải được reserved trong Core area; Hard IP height phải là bội số của Site height để không bisect Rows; OBS extent của Hard IP ảnh hưởng đến routing channel availability; Macro placement là một trong những critical decisions trong Floorplanning
- **[[Routing]]**: Hard IP có large OBS regions (thường M1–M4 hoặc toàn bộ layers) — Router phải route signals vào Hard IP Pins từ trên OBS extent (ví dụ M5+); Hard IP Pin locations (từ LEF) là fixed routing targets không thể move
- **[[STA]]**: Hard IP LIB file chứa timing models của IP interface pins — Setup/Hold checks tại input Pins và Output Delay specs tại output Pins; Source Latency của Hard IP PLL output là input cho SDC clock latency modeling
- **[[Signoff]]**: Hard IP GDS được merge vào chip-level GDS; DRC của Hard IP internal là IP vendor's responsibility nhưng interface DRC (spacing between Hard IP boundary và neighboring cells/wires) là PD engineer's responsibility; LVS compare Hard IP CDL với Hard IP GDS

## Requires
- [[LEF]] — Cell Abstract (Physical Abstract) của Hard IP là bắt buộc cho PnR; không có LEF, Placement tool không biết IP size và Router không biết Pin locations và OBS
- [[LIB]] — Timing model của Hard IP interface pins là bắt buộc cho STA; MMMC cần LIB per corner cho Hard IP cũng như Standard Cells
- [[GDS]] — Full layout của Hard IP; merge vào chip-level GDS cho DRC/LVS và Tape-out
- [[CellAbstract]] — Hard IP LEF là một dạng Cell Abstract; cùng format nhưng thường với larger SIZE và more extensive OBS than Standard Cells

## Used by
- [[Floorplanning]] — Hard IP placement là first-priority trong Floorplanning; vị trí SRAM, PHY, PLL được locked trước khi Standard Cell placement bắt đầu; Hard IP locations ảnh hưởng đến toàn bộ chip topology
- [[Placement]] — Hard IP instances được placed với FIXED status — Placement tool không được move chúng; Standard Cells được placed around Fixed Hard IP instances
- [[Routing]] — Hard IP Pins là fixed routing targets; Router phải route signals đến Hard IP Pins đi qua OBS-free layers; hard IP boundary spacing DRC phải be met
- [[STA]] — Hard IP LIB timing models được included trong MMMC setup; paths through Hard IP (input-to-output) được characterized và included trong timing analysis; PLL output clock từ Hard IP là clock source cho create_clock trong SDC
- [[Signoff]] — Hard IP GDS merged vào chip GDS; interface DRC checked; LVS includes Hard IP CDL

## Key insight
[USER REVIEW — draft suggestion]: Hard IP integration là một trong những skills phân biệt junior và senior PD engineer — nó đòi hỏi hiểu biết về cả analog/memory design (để đặt đúng questions cho IP vendor) lẫn digital PD (để integrate đúng vào flow). Ba vấn đề phổ biến nhất khi integrate Hard IP: (1) IP LIB không cover đủ corners → STA thiếu corners → potential silicon failure; (2) IP LEF có OBS không chính xác → routing shorts hoặc over-conservative routing channels; (3) IP Pin locations không align với Routing Grid → routing difficulty tăng đột biến. Đây là lý do IP qualification (verify đầy đủ và chính xác của tất cả views) là mandatory step trước khi bắt đầu PnR flow với Hard IP. Memory Compiler là exception — nó auto-generate tất cả views nên quality thường đảm bảo; vấn đề hay gặp là chọn sai Memory Compiler configuration (wrong word count, bit width) dẫn đến chip-level integration issues.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Contrasted with: Soft IP (RTL-based, goes through synthesis)
→ Views required: [[LEF]] · [[LIB]] · [[GDS]] · Verilog · CDL/SPICE
→ Contains: [[CellAbstract]] (LEF), large [[Obstruction]] regions
→ Examples: SRAM · ROM · PLL · PHY · ADC/DAC
→ Generated by: Memory Compiler (memory) · Custom design (analog)
→ Critical for: [[Floorplanning]] topology · [[Routing]] channel planning · [[STA]] clock modeling
→ Cùng nhóm: [[StandardCell]] · [[CellAbstract]] · [[LEF]] · [[LIB]] · [[GDS]]