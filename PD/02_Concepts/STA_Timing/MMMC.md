---
tags: [concept, sta-timing]
group: STA — Timing
defined_in: MMMC TCL file — manually written, typically by Synthesis Engineer
used_by: [STA, Placement, ClockTreeSynthesis, Routing, Signoff, DesignImport]
requires: [LIB, SDC, ITF]
chain: Chain_STA_Basics
---
# MMMC

## Definition
Multi-Mode Multi-Corner (MMMC) là framework tổ chức toàn bộ không gian phân tích Timing của một design thành 5 lớp phân cấp lồng nhau. Mỗi lớp encapsulate một tập thông tin timing cụ thể, và các lớp được combine từ dưới lên để tạo ra các Analysis Views — các "lens" mà STA engine nhìn vào design.

**Kiến trúc 5 lớp**:

Library Set          RC Corner
(PVT + LIB files)   (qrcTechFile + temperature)
↓                   ↓
└──── Delay Corner ─┘
(cell timing + wire parasitic)
↓
Constraint Mode (SDC + operating mode)
↓
Analysis View
(full timing context cho một scenario)

| Lớp | Định nghĩa | Số lượng điển hình |
|---|---|---|
| Library Set | Tập LIB files characterize cells tại một PVT corner | 3–6 sets |
| RC Corner | Điều kiện extraction RC (qrcTechFile + temperature) | 2–3 corners |
| Delay Corner | Library Set + RC Corner — xác định total path delay | 4–8 corners |
| Constraint Mode | Một operating mode với SDC file riêng | 2–4 modes |
| Analysis View | Delay Corner + Constraint Mode — một complete timing scenario | 8–40 views |

Active Analysis Views là subset được STA engine thực sự analyze trong mỗi PnR stage. Setup analysis dùng view có `dc_max`; Hold analysis dùng view có `dc_min`.

MMMC file là TCL file, được đọc bởi PnR tool để khởi tạo toàn bộ timing infrastructure trước Floorplanning.

## Computed from

**Library Set** — một set tương ứng một PVT corner. Các PVT corners điển hình:

| Corner | Process | Voltage | Temp | Dùng cho |
|---|---|---|---|---|
| SSH (Slow-Slow-Hot) | $N_{slow}\ P_{slow}$ | $V_{min}$ | $T_{max}$ (125°C) | Setup — max cell delay |
| FFH (Fast-Fast-Hot) | $N_{fast}\ P_{fast}$ | $V_{max}$ | $T_{max}$ | Hold tại hot temp |
| FFC (Fast-Fast-Cold) | $N_{fast}\ P_{fast}$ | $V_{max}$ | $T_{min}$ (−40°C) | Hold — min cell delay |
| TT (Typical-Typical) | Nominal | $V_{nom}$ | $T_{nom}$ | Reference |

**RC Corner** — mỗi corner tương ứng một qrcTechFile và temperature:
- `rc_max`: high temperature → max metal resistance (Cu resistance tăng theo nhiệt độ) + max capacitance → max Net Delay
- `rc_min`: low temperature → min metal resistance + min capacitance → min Net Delay

**Delay Corner** — pairing có chủ ý giữa Library Set và RC Corner:
- `dc_max` = SSH + rc_max: slow transistors + hot + max wire RC → **worst-case Setup** (maximum total path delay)
- `dc_min` = FFC + rc_min: fast transistors + cold + min wire RC → **worst-case Hold** (minimum total path delay)

Lý thuyết đằng sau pairing: Setup cần maximum total delay (cả cell và wire phải worst); Hold cần minimum total delay (cả cell và wire phải best). Pairing `dc_max` cho Setup và `dc_min` cho Hold là worst-case scenario — design pass cả hai sẽ pass mọi corner trung gian.

**Constraint Mode** — một operating mode, ví dụ:
- `cm_func`: Functional mode (normal chip operation) với func.sdc
- `cm_scan`: Scan test mode với scan.sdc (scan chains active, timing khác với functional)

**Analysis View** = Delay Corner + Constraint Mode. Ví dụ:
- `av_func_max`: Functional mode + max delay → dùng cho Setup check trong functional operation
- `av_scan_min`: Scan mode + min delay → dùng cho Hold check trong scan test

**Active Analysis Views** phải được chỉ định tường minh — Setup checks và Hold checks có thể dùng different Analysis Views để reflect real-world worst-case scenarios.

## Constrains
- **[[STA]]**: Slack của mọi path được tính trên Active Analysis Views; số lượng Analysis Views quyết định wall-clock time của mỗi STA run
- **[[Signoff]]**: phải pass toàn bộ Analysis Views trước Tape-out; thêm views = tăng coverage nhưng cũng tăng runtime

## Requires
- [[LIB]] — timing model nguồn cho Library Sets (một LIB file per cell type per PVT corner)
- [[SDC]] — constraint files nguồn cho Constraint Modes
- [[ITF]] — compiled thành binary qrcTechFiles; mỗi qrcTechFile được reference trong một RC Corner

## Used by
- [[STA]] — engine chạy analysis trên từng Active Analysis View đồng thời
- [[Placement]], [[ClockTreeSynthesis]], [[Routing]] — optimization engines đọc MMMC setup/hold views để guide timing-driven decisions
- [[Signoff]] — full MMMC STA là điều kiện bắt buộc trước Tape-out
- [[DesignImport]] — MMMC file là input thứ 3 của Import step

## Key insight
[USER REVIEW — draft suggestion]: MMMC là nguyên nhân chính khiến STA runtime của một production chip có thể mất nhiều giờ — 20–40 Analysis Views, mỗi view traverse hàng triệu paths. Tradeoff giữa coverage (nhiều views = phát hiện nhiều corner violations) và runtime (nhiều views = chậm hơn) là quyết định kiến trúc của project. Một mistake phổ biến của người mới: focus vào worst-case Setup (dc_max) nhưng neglect worst-case Hold (dc_min). Hold violations phát sinh sau CTS và không thể giải quyết bằng frequency reduction — chúng là silicon killers nếu không được detect và fix trước Tape-out.

## Related
→ Chain: [[Chain_STA_Basics]]
→ 5-layer hierarchy: Library Set → RC Corner → Delay Corner → Constraint Mode → Analysis View
→ Critical pairing: dc_max (SSH + rc_max) cho Setup · dc_min (FFC + rc_min) cho Hold
→ Source files: [[LIB]] · [[SDC]] · [[ITF]] (qrcTechFile)
→ Consumed by: [[STA]] · [[Placement]] · [[ClockTreeSynthesis]] · [[Routing]] · [[Signoff]] · [[DesignImport]]
→ Cùng nhóm: [[STA]] · [[SDC]] · [[LIB]] · [[ITF]] · [[Slack]]