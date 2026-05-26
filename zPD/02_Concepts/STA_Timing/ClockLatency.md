---
tags: [concept, sta-timing, clock]
group: STA — Clock
defined_in: STA tool (measured post-CTS) · SDC (modeled pre-CTS via set_clock_latency)
used_by: [STA, ClockTreeSynthesis, SDC, Signoff]
requires: [ClockTreeSynthesis, SDC]
chain: Chain_STA_Basics
---
# ClockLatency

## Definition
Clock Latency (a.k.a. Clock Insertion Delay) là tổng thời gian tín hiệu clock đi từ điểm nguồn phát ban đầu đến chân clock (clock pin) của một Flip-flop cụ thể. Clock Latency là con số độc lập cho từng FF — không phải một giá trị chung cho toàn bộ clock domain. Latency cao không nhất thiết là vấn đề; vấn đề xảy ra khi Latency của các FF trong cùng domain chênh lệch nhau lớn (tức là Clock Skew lớn). Latency gồm hai thành phần cộng lại: Source Latency và Network Latency.

## Computed from
$$\text{Total Latency}_{FF} = \text{Source Latency} + \text{Network Latency}$$

**Source Latency**: Thời gian từ điểm gốc sinh ra clock (thường là PLL hoặc crystal oscillator bên ngoài chip) đến Clock Port của block/macro đang thiết kế. Source Latency nằm ngoài tầm kiểm soát của PD engineer — nó phụ thuộc vào board-level routing và PLL characteristics.

**Network Latency**: Thời gian từ Clock Port đi xuyên qua toàn bộ mạng buffer của Clock Tree để đến clock pin của FF bên trong block. Network Latency = kết quả của CTS — đây là phần PD engineer kiểm soát được.

Pre-CTS: STA dùng `set_clock_latency` trong SDC để model estimated Latency cho cả Launch và Capture paths — giúp synthesis và pre-CTS placement có timing estimate hợp lý. Post-CTS: `set_propagated_clock` thay thế — STA compute Latency thực tế từ Clock Tree buffer chain với actual RC delays.

## Constrains
- **[[ClockSkew]]**: Skew = hiệu số Latency giữa 2 FF; Latency là thành phần cơ bản tạo nên Skew — không thể hiểu Skew mà không hiểu Latency per-FF
- **[[SetupTime]]**: Launch Clock Latency cộng vào AT của Launch path; Capture Clock Latency cộng vào RAT của Capture path; hiệu số giữa hai Latency = Clock Skew ảnh hưởng đến [[Slack]]
- **[[ClockTreeSynthesis]]**: Minimizing Network Latency variation giữa các FF là primary objective của CTS bên cạnh minimizing Global Skew; Latency budget được specify trong CTS constraints

## Requires
- [[ClockTreeSynthesis]] — Network Latency là output của CTS; trước CTS chỉ có estimate; sau CTS mới có actual Latency per-FF
- [[SDC]] — `set_clock_latency [-source] value [get_clocks clk]` model pre-CTS Latency; `-source` flag chỉ Source Latency; không có flag chỉ Network Latency; sau CTS thay bằng `set_propagated_clock`

## Used by
- [[STA]] — Clock Latency được annotate lên Launch và Capture clock paths trong timing graph; Launch Latency ảnh hưởng đến AT; Capture Latency ảnh hưởng đến RAT; hiệu số = Skew được fold vào Slack equations
- [[ClockTreeSynthesis]] — CTS tool minimize variation trong Network Latency giữa tất cả FFs; target Latency range được set trong CTS spec (ví dụ: max Skew < 50ps tương đương max Latency variation < 50ps)
- [[Signoff]] — post-route Signoff STA dùng propagated clock với full SPEF để compute final Latency per-FF; đây là con số chính xác nhất và là basis cho Timing Signoff

## Key insight
[USER REVIEW — draft suggestion]: Sự phân chia Source Latency / Network Latency quan trọng trong hierarchical design flow — khi một block được thiết kế độc lập (block-level PD), PD engineer của block đó không biết Source Latency (từ PLL đến block boundary) là bao nhiêu. Họ phải dùng `set_clock_latency -source` để model ước tính giá trị này. Khi block được integrated vào chip-level, chip-level team sẽ override giá trị này bằng actual routing delay. Nếu Source Latency estimate ở block-level sai lệch nhiều so với chip-level actual, timing correlation sẽ kém — đây là nguồn gốc phổ biến của "timing surprises" trong chip integration.

## Related
→ Chain: [[Chain_STA_Basics]]
→ Formula: Total Latency = Source Latency + Network Latency
→ Produced by: [[ClockTreeSynthesis]] (Network Latency)
→ Modeled by: [[SDC]] — set_clock_latency · set_propagated_clock
→ Enables: [[ClockSkew]] computation
→ Related: [[ClockSkew]] · [[ClockUncertainty]] · [[ClockJitter]]
→ Cùng nhóm: [[ClockSkew]] · [[ClockUncertainty]] · [[ClockJitter]] · [[STA]] · [[SDC]]