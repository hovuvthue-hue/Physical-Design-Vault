---
tags: [concept, signoff, reliability]
group: Signoff
defined_in: Reliability / Signoff — physical wear-out mechanism in metal interconnect
used_by: [Signoff, Routing, Floorplanning, PDN]
requires: [MetalStack, Routing, PDN, IRDrop]
chain: Chain_PnR_Flow
---
# Electromigration

## Definition
Electromigration (EM) là sự dịch chuyển dần dần của các nguyên tử kim loại trong dây dẫn của IC khi mật độ dòng điện đủ cao để gây ra drift của metal ions theo chiều chuyển động của electron. EM là **wear-out mechanism** — không gây lỗi ngay lập tức mà tích lũy theo thời gian vận hành, dẫn đến chip failure ngoài hiện trường.

## Physical mechanism

Electron chuyển động từ cathode sang anode. Khi mật độ dòng cao, electron transfer momentum cho metal atoms → nguyên tử kim loại bị đẩy theo chiều electron flow → vật liệu dịch chuyển khỏi vùng cathode, tích tụ tại vùng anode:

| Vùng | Hiện tượng | Failure mode |
|---|---|---|
| **Cathode** | Thiếu hụt nguyên tử → hình thành **Void** | Tăng resistance dần → **Open circuit** (interconnect failure) |
| **Anode** | Tích tụ nguyên tử → hình thành **Hillock** | Phình ra → **Short circuit** với wire lân cận |

## Causes

**1. High DC current density:**
Nguyên nhân chính. Foundry/PDK quy định $J_{max}$ (maximum current density) cho từng metal layer và via type. [Needs verification per PDK]

**2. Joule heating từ alternating current:**
Dòng AC tần số cao gây Joule heating. Nhiệt làm tăng tốc độ dịch chuyển nguyên tử kim loại.

**3. Thermal cycling:**
Pulses chạy qua wire → nhiệt phát sinh → wire nóng vượt nhiệt độ oxide → chênh lệch thermal constants giữa oxide và metal gây mechanical stress lặp lại → wire failure dẫn đến chip failure.

## EM vs IR Drop

Cả EM và [[IRDrop]] đều liên quan đến current trong PDN và signal wires, nhưng khác nhau về thời gian tác động:

| | IR Drop | Electromigration |
|---|---|---|
| Tác động | Ngay lập tức (voltage sag khi có dòng) | Tích lũy theo thời gian (wear-out) |
| Failure mode | Timing violation | Physical failure (open/short) |
| Fix stage | Floorplanning / Routing | Design rule enforcement tại Signoff |

## EM trong Signoff

EM analysis là một phần bắt buộc của [[Signoff]], thường chạy cùng IR Drop analysis (EMIR). Tool: Cadence Voltus, Synopsys RedHawk. Ngưỡng pass/fail ($J_{max}$ per layer, per via, per temperature) phụ thuộc foundry, PDK, operating temperature, và product lifetime target. [Needs verification]

## Mitigation strategies (concept level)

- **Wider wires**: tăng cross-sectional area $A$ → giảm current density $J = I/A$
- **Higher metal layers**: thick/ultra-thick layers trong [[MetalStack]] có higher current capacity và lower sheet resistance
- **Via redundancy / multi-cut via**: phân tán current qua nhiều via cut → giảm $J$ tại điểm chuyển layer; policy cụ thể là tool/PDK/foundry-specific [Needs verification]
- **PDN width planning**: tránh narrow choke points trong power grid tập trung current

## Requires
- [[MetalStack]] — layer thickness và resistivity ảnh hưởng trực tiếp đến EM susceptibility
- [[Routing]] — routed wire geometry quyết định current density distribution
- [[PDN]] — power distribution là nguồn high DC current trong chip
- [[IRDrop]] — EMIR analysis thường chạy cùng nhau; IR drop tăng khi void tích lũy làm resistance tăng theo thời gian

## Used by
- [[Signoff]] — EM là một signoff gate cùng với DRC/LVS/Timing/IR Drop
- [[Routing]] — layer assignment phải tránh $J_{max}$ violation trên critical/power nets
- [[Floorplanning]] — PDN design phải account for EM budget khi chọn stripe width và layer assignment
- [[PDN]] — power grid topology ảnh hưởng trực tiếp đến current distribution và EM risk

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Co-analysis: [[IRDrop]] (cùng nhóm EMIR)
→ Failure modes: Void (open circuit) · Hillock (short circuit)
→ Signoff tools: Cadence Voltus · Synopsys RedHawk
→ Layer context: [[MetalStack]] · [[Via]]
→ Cùng nhóm: [[IRDrop]] · [[PDN]] · [[Signoff]] · [[DFM]]