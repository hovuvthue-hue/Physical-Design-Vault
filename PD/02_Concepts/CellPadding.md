---
tags: [concept, pnr-flow, placement]
group: PnR Flow
defined_in: Post-Placement Optimization — Congestion reduction technique targeting high pin density cells
used_by: [Placement, CongestionAnalysis, PreCTSOptimization]
requires: [Placement, StandardCell, PlacementBlockage]
chain: Chain_PnR_Flow
---
# CellPadding

## Definition
**Cell Padding** là kỹ thuật thêm spacing constraint xung quanh các cell types cụ thể trong quá trình Placement, nhằm giảm local pin density và cải thiện routability. Khác với [[PlacementBlockage]] áp dụng cho một vùng layout xác định, Cell Padding gắn với một loại cell cụ thể theo tên trong library và đi theo từng instance của cell đó tại mọi vị trí trong design.

## Why Cell Padding is needed
Một số Standard Cell types có số lượng Pin trên mỗi cell rất lớn — đặc biệt là Flip-flop, [[MBFF]], và complex gate phức tạp. Khi Placement engine đặt nhiều instances của các cells này liền kề nhau, mật độ Pin trong vùng đó có thể quá cao, dẫn đến:
- [[PinAccess]] khó — Router gặp khó khăn khi tiếp cận từng Pin riêng biệt trong không gian chật hẹp.
- Routing congestion cục bộ — quá nhiều nets phải route trong một vùng nhỏ.
- Nguy cơ max transition / max capacitance DRV violations do load tập trung.

Cell Padding giải quyết bằng cách tạo vùng spacing buffer bao quanh mỗi cell instance, buộc Placement engine phải giữ khoảng cách tối thiểu với các cells xung quanh.

## Mechanism
Cell Padding tạo một placement blockage nhỏ gắn liền với từng cell instance của type được target. Vùng này có thể được chỉ định riêng cho từng phía (trái/phải/trên/dưới) của cell instance với kích thước khác nhau. Spacing constraints được áp dụng vào library cells qua tool, không phải vào layout regions.

Về bản chất: Cell Padding là placement blockage cục bộ gắn với cell — khác PlacementBlockage ở chỗ nó di chuyển cùng cell instance thay vì cố định trong layout.

## Primary targets
Cell Padding thường được áp dụng cho:
- **Flip-flop và MBFF** — số lượng lớn, pin count cao (D, CK, Q, SE, SI và nhiều data pins với MBFF).
- **Complex gate** có fanin cao.
- **Bất kỳ cell type nào** mà khi cluster dày đặc tạo ra local congestion hoặc DRV risk.

## Relation to PlacementBlockage
[[PlacementBlockage]] và Cell Padding đều dùng cơ chế blockage nhưng ở phạm vi khác nhau:

| | [[PlacementBlockage]] | CellPadding |
|---|---|---|
| Phạm vi | Region-based (vùng layout cố định) | Cell-based (theo tên cell trong library) |
| Di chuyển khi cell di chuyển? | Không | Có — đi theo cell instance |
| Use case chính | Macro keep-out, congestion hotspot vùng | High pin density cell types |

Hai kỹ thuật bổ trợ nhau: PlacementBlockage xử lý congestion theo vùng địa lý, CellPadding xử lý theo nguồn gốc cell type.

## Trade-offs
- Cell Padding làm tăng effective cell footprint → placement utilization tăng → có thể cần tăng Core Area hoặc giảm target utilization.
- Cell Padding quá aggressive → dồn ép các cells còn lại → có thể tạo hotspot mới tại vùng khác.
- Cell Padding không cần thiết → lãng phí area.

## Position in flow
Cell Padding được thiết lập trong giai đoạn **post-placement optimization** khi [[CongestionAnalysis]] phát hiện hotspot gắn với các cell types có pin density cao. Thường được setup trước khi chạy lại placement hoặc trong vòng lặp refine.

## Requires
- [[Placement]]
- [[StandardCell]]
- [[PlacementBlockage]]

## Used by
- [[Placement]] — placement engine enforce spacing constraints trong quá trình legalization.
- [[CongestionAnalysis]] — Cell Padding là một trong các biện pháp giảm pin-density congestion.
- [[PreCTSOptimization]] — có thể áp dụng trong pre-CTS pass để cải thiện routability.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Closely related: [[PlacementBlockage]] · [[CongestionAnalysis]] · [[PlacementDensity]] · [[MBFF]]
→ Cùng nhóm: [[Placement]] · [[PlacementBlockage]] · [[CongestionAnalysis]]