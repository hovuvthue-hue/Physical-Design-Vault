---
tags: [concept, pnr-flow, placement, power, leakage]
group: PnR Flow
defined_in: PowerAnalysis
used_by: [PowerAnalysis, PowerOptimization, Signoff]
requires: [StandardCell, PowerAnalysis]
chain: Chain_PnR_Flow
---
# LeakagePower

## Definition
**LeakagePower** / **Static Power** là phần power vẫn bị tiêu thụ ngay cả khi cell không switching, do các dòng rò không lý tưởng bên trong transistor.

Ở mức concept, tổng power thường được tách thành:

**Total Power = Static Power + Dynamic Power**

LeakagePower khác với **Dynamic Power** vì nó không cần signal transition để phát sinh. Trong implementation flow, leakage thường được phân tích trong context của library, process, corner, activity/power-analysis setup và methodology; giá trị chính xác phụ thuộc library, PVT và flow đo/ước lượng cụ thể. [Needs verification]

## Main leakage components
LeakagePower không phải một cơ chế duy nhất. Nó là tập hợp nhiều leakage current sources, trong đó các thành phần dưới đây thường được nhắc đến ở mức concept.

### Subthreshold Leakage Current
**Subthreshold Leakage Current** xảy ra khi gate voltage thấp hơn threshold voltage **Vth**, nhưng vẫn còn một dòng dẫn yếu từ drain sang source khi transistor được kỳ vọng là OFF.

Lower Vth giúp transistor switching nhanh hơn nhưng làm subthreshold leakage tăng. Đây là **major contributor** trong scaled technologies — ở các process nodes nhỏ, việc giảm Vth để duy trì performance đồng nghĩa với subthreshold leakage tăng đáng kể.

### Gate Leakage Current
**Gate Leakage Current** phát sinh do tunneling qua gate oxide khi điện trường qua oxide đủ mạnh.

Gate oxide mỏng hơn có thể làm tăng rủi ro tunneling, nhưng magnitude thực tế phụ thuộc device, process và oxide stack. [Needs verification]

### Reverse-Biased Junction Leakage
**Reverse-Biased Junction Leakage** xảy ra tại các PN junction giữa source/drain và substrate khi junction bị reverse bias.

Ở mức minh họa concept, NMOS source/drain ở mức gần **VDD** hoặc PMOS source/drain ở mức gần **VSS** có thể tạo context reverse-biased junction so với substrate/well tương ứng. Cách bias cụ thể phụ thuộc cell topology, well/substrate connection và logic state, nên không nên suy rộng thành cùng một trường hợp cho mọi standard cell. [Needs verification]

### Gate-Induced Drain Leakage / GIDL
**Gate-Induced Drain Leakage (GIDL)** là leakage liên quan đến điện trường mạnh gần vùng gate-drain.

Trong L8, GIDL được liệt kê như một thành phần leakage bên cạnh diode reverse-bias current, subthreshold current và gate oxide leakage. Mức độ ảnh hưởng của GIDL là process/device-specific. [Needs verification]

## Factors affecting leakage

| Factor                        | Concept-level effect                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------ |
| ↓ Threshold Voltage           | ↑ Subthreshold leakage.                                                              |
| ↑ Supply Voltage              | ↑ Gate & junction leakage do điện trường/bias lớn hơn.                               |
| ↑ Temperature                 | ↑ All leakage components.                                                            |
| ↑ Transistor Width            | ↑ Leakage (larger area — nhiều leakage paths hơn).                                  |
| ↓ Oxide Thickness             | ↑ Gate oxide tunneling.                                                              |
| Reverse Body Bias             | ↓ Leakage (tăng effective Vth). [Needs verification — phụ thuộc architecture/PDK]   |
| Forward Body Bias             | ↑ Leakage (giảm effective Vth). [Needs verification — phụ thuộc architecture/PDK]   |
| Process Variation             | Variable leakage per device/cell instance.                                           |
| Logic State / Stacking effect | Stacking effect → ↓ leakage (stacked OFF devices reduce subthreshold current path). |

Bảng tương ứng cho dynamic power factors: xem [[PowerAnalysis]].
## Technology scaling trend
Ở các technology nodes lớn hơn (cũ hơn), **Dynamic Power** là phần dominant trong tổng power. Khi technology scaling đi xuống, LeakagePower tăng đáng kể và trở thành thành phần quan trọng trong tổng power ở advanced nodes.

Các cơ chế cụ thể:
- Lower supply voltage → reduced Vth → higher subthreshold leakage.
- Thinner gate oxide → increased gate tunneling current.
- Short-channel effects trở nên prominent hơn.

Tỷ trọng thực tế giữa leakage và dynamic power phụ thuộc process, library, VDD, temperature và workload/activity của design.

## Relation to StandardCell Vt
Các lựa chọn **LVT / SVT / HVT** trong [[StandardCell]] là một lever quan trọng để cân bằng leakage và timing.

- **Lower Vt** thường cải thiện speed nhưng tăng LeakagePower.
- **Higher Vt** thường giảm LeakagePower nhưng có thể làm timing khó hơn.
- Multi-Vt standard-cell libraries cho phép [[PowerOptimization]] trade off giữa leakage và timing bằng cell swap / Vt swap khi điều kiện timing cho phép.

Không nên giả định mọi Vt variant luôn có cùng footprint, pin access hoặc legal equivalence nếu chưa được library/PDK xác nhận. [Needs verification]

## Relation to PowerOptimization
Leakage optimization có thể dùng cell swap, Vt swap hoặc drive-strength choice khi [[Slack]] và các ràng buộc implementation cho phép.

Việc giảm leakage phải được kiểm soát bởi [[STA]], DRV, placement legality, library equivalence và signoff context. Nếu thay cell sang HVT hoặc downsizing quá mạnh, path có thể chậm hơn hoặc tạo rủi ro slew/load mới; vì vậy PowerOptimization không nên được mô tả như một bước luôn giảm leakage mà không ảnh hưởng timing. [Needs verification]

## Requires
- [[StandardCell]]
- [[PowerAnalysis]]

## Used by
- [[PowerAnalysis]]
- [[PowerOptimization]]
- [[Signoff]]

## Related
- [[PowerAnalysis]]
- [[PowerOptimization]]
- [[StandardCell]]
- [[STA]]
- [[Slack]]
