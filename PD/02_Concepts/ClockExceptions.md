---
tags: [concept, pnr-flow, cts, clock]
group: PnR Flow
defined_in: ClockTreeSynthesis — CTS exception configuration
used_by: [ClockTreeSynthesis, CTSOptimization, CTSQualityReview]
requires: [ClockTreeSynthesis, ClockSkewGroup]
chain: Chain_PnR_Flow
---
# ClockExceptions

## Definition
CTS Exceptions là các clock pins hoặc endpoints đặc biệt cần cách xử lý tùy chỉnh trong quá trình clock tree construction và optimization. CTS tool có thể không tự động nhận diện hoặc xử lý đúng các pins này như normal sink pins. Designer phải tường minh chỉ định cho CTS tool cách treat từng exception pin.

## Why CTS Exceptions Exist
Trong design phức tạp, không phải mọi clock pin đều nên được CTS xử lý như nhau:
- Một số pins là điểm phân nhánh của clock topology (ICG, clock mux) — CTS không nên insert buffer vào đó
- Một số pins không cần tham gia skew balancing nhưng clock vẫn cần đến đó
- Một số pins hoàn toàn nằm ngoài scope của CTS optimization
- Một số pins không có physical clock path nhưng cần được model cho mục đích balancing

CTS exceptions cho phép designer kiểm soát chính xác hành vi CTS tại từng loại pin đặc biệt.

## Common CTS Exception Pin Types

**Stop Pin:**
Stop pin là endpoint của clock tree nơi CTS dừng clock propagation và thực hiện delay balancing. CTS treat stop pin là valid clock sink endpoint cho: skew balancing, insertion delay optimization, và clock latency control.

Có hai loại stop pin:

- **Implicit Stop Pins**: CTS tool tự động nhận diện là valid clock sink endpoints. Bao gồm flip-flop clock pins, latch clock pins, và macro clock pins. CTS tự động dừng clock tree construction tại các điểm này mà không cần designer chỉ định tường minh.

- **Explicit Stop Pins**: do designer tường minh chỉ định. Thường dùng cho: hierarchical CTS, generated clocks, macro boundaries, và custom clock structures.

Xác định đúng stop pins giúp: cải thiện skew balancing, cải thiện timing closure, kiểm soát insertion delay, và tránh clock buffering không cần thiết.

**Nonstop Pin:**
CTS nhìn xuyên qua cell (transparent) — CTS kiểm soát hoàn toàn toàn bộ clock path liên tục qua pin này, có đầy đủ tự do điều chỉnh tất cả segments khi balancing. Clock đi vào qua ICG input pin và đi ra qua ICG output pin; tất cả segments được CTS route. Thường dùng cho ICG input clock.

**Exclude Pin:**
CTS không kết nối pin này vào clock tree và không route clock đến pin này — pin bị bỏ qua hoàn toàn trong timing analysis của clock tree. Dùng cho các pins được dùng làm clock trong scan/test mode; tần số scan rất chậm nên DRV signal thông thường là đủ, không cần CTS xử lý.

**Float Pin:**
Float pin là clock sink pin với user-defined insertion delay target. Float pin được dùng khi macro/memory/hard IP block có internal clock path — tức là clock phải đi thêm một đoạn bên trong block sau khi chạm đến chân clock pin vật lý. CTS model thêm đoạn internal delay này để đảm bảo balancing chính xác với các sinks còn lại.

Float pin thường được dùng cho: macros, memories, hard IP blocks, và hierarchical clock interfaces.

Cơ chế tính toán:

$$\text{Float pin delay} = \text{Target insertion delay} - \text{Internal delay of macro/memory}$$

Ví dụ: Target insertion delay = 2.0 ns, Memory internal insertion delay = 0.2 ns → Float pin delay = 1.8 ns. CTS xây dựng clock tree sao cho insertion delay đến chân clock pin của memory là 1.8 ns; cộng thêm 0.2 ns internal delay bên trong memory, tổng insertion delay đạt đúng target 2.0 ns.

**Ignore Pin:**
CTS vẫn kết nối pin này vào clock tree và vẫn route clock đến pin này, nhưng pin không tham gia skew balancing. Pin nhận clock nhưng không ảnh hưởng đến thuật toán cân bằng skew. Dùng cho các pins có non-critical timing hoặc unusual insertion delay.

**Through Pin:**
CTS coi cell này là grey box — không biết implementation vật lý bên trong, nhưng biết delay từ CLK_IN đến CLK_OUT từ .lib. Clock đi vào qua macro input pin và đi ra qua macro output pin; segments bên ngoài macro do CTS route, segment nội bộ do macro tự route. Fixed .lib delay được cộng vào insertion delay; CTS chỉ điều chỉnh segments bên ngoài. Yêu cầu propagated clock được khai báo trên macro CLK_OUT pin và .lib arc phải được characterized. Thường dùng cho feedthrough macro / hard IP.

## Key Distinction: Exclude vs Ignore

|                                      | Exclude Pin              | Ignore Pin                                       |
| ------------------------------------ | ------------------------ | ------------------------------------------------ |
| Kết nối vào clock tree?              | Không                    | Có                                               |
| CTS route clock đến pin?             | Không                    | Có                                               |
| Tham gia skew balancing?             | Không                    | Không                                            |
| Tham gia clock tree timing analysis? | Không                    | Không                                            |
| Use case                             | Clock cho scan/test mode | Non-critical timing hoặc unusual insertion delay |

## Key Distinction: Nonstop vs Through

|                             | Nonstop Pin                             | Through Pin                                                              |
| --------------------------- | --------------------------------------- | ------------------------------------------------------------------------ |
| CTS visibility inside cell? | Transparent — CTS controls full path    | Grey box — chỉ biết CLK_IN→CLK_OUT delay từ .lib                         |
| Clock routing               | Tất cả segments do CTS route            | Segments ngoài macro: CTS route; nội bộ: macro tự route                  |
| Skew balancing freedom      | Đầy đủ — CTS điều chỉnh tất cả segments | Giới hạn — CTS chỉ điều chỉnh segments ngoài macro                       |
| Extra constraints needed?   | Không                                   | Có — propagated clock trên CLK_OUT pin; .lib arc phải được characterized |
| Typical use                 | ICG input clock                         | Feedthrough macro / hard IP                                              |
## Relation to ClockSkewGroup
CTS exceptions ảnh hưởng đến phạm vi của [[ClockSkewGroup]]:
- Stop/Ignore pins giới hạn phạm vi sinks trong skew group
- Nonstop/Through pins mở rộng hoặc định hướng clock propagation qua các điểm trung gian
- Exception configuration phải nhất quán với skew group definitions

## Requires
- [[ClockTreeSynthesis]] — exceptions phải được configure trước khi CTS chạy
- [[ClockSkewGroup]] — exception types ảnh hưởng đến cách sinks được phân bổ và scope của từng group

## Used by
- [[ClockTreeSynthesis]] — CTS tool đọc exception definitions và apply khi build clock tree
- [[CTSOptimization]] — optimization scope bị định hướng bởi exception configuration
- [[CTSQualityReview]] — review phải xác nhận exception handling đúng với intent của designer

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Exception types: Stop · Nonstop · Exclude · Floating · Ignore · Through
→ Key distinction: Exclude (clock propagates, no balancing) vs Ignore (completely excluded)
→ Closely related: [[ClockTreeSynthesis]] · [[ClockSkewGroup]] · [[CTSOptimization]] · [[ICG]]
→ Cùng nhóm: [[ClockTreeSynthesis]] · [[ClockSkewGroup]] · [[ICG]] · [[CTSFlow]]