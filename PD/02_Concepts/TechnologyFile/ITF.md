---
tags: [concept, technology-files]
group: Technology Files
defined_in: Foundry — provided as part of PDK (Process Design Kit)
used_by: [MMMC, ParasiticExtraction, DesignImport]
requires: [MetalStack]
chain: Chain_LEF_to_PnR
---
# ITF

## Definition
Interconnect Technology File (ITF) chứa các thông số vật liệu R/C của từng metal layer cho một RC corner cụ thể. Ở mức khái niệm, ITF là nguồn dữ liệu công nghệ để [[ParasiticExtraction]] tính [[InterconnectRC]] và ghi kết quả ra [[SPEF]]. ITF là input cho tool **Techgen** để compile ra binary RC tech files (qrcTechFiles) — sau đó được dùng bởi hai consumers độc lập: P&R tool (Innovus) và RC Extraction tool (QRC). Cả hai đều output RC values dưới dạng **SPEF**.

ITF tồn tại ở hai dạng phân biệt theo vòng đời:
- **Text format (.ICT)**: human-readable, do Foundry cung cấp trong PDK; được Techgen đọc để compile
- **Binary format (qrcTechFile)**: compiled output của Techgen, dùng trực tiếp bởi PnR và RC Extraction tools

PD engineer chỉ tương tác với binary format. Text format thuộc về Foundry và CAD team khi update PDK hoặc debug R/C values.

**Tại sao cần nhiều qrcTechFiles?** RC values phụ thuộc vào nhiệt độ và manufacturing variation:
- `qrcTechFile_rcworst`: max resistance + max capacitance (high temperature, thick/wide wires as per process variation) → dùng cho worst-case Setup analysis (max Net Delay)
- `qrcTechFile_rcbest`: min resistance + min capacitance (low temperature, thin/narrow wires) → dùng cho worst-case Hold analysis (min Net Delay)
- `qrcTechFile_typical`: nominal conditions

Pipeline đầy đủ:

$$\underbrace{\text{Foundry}}_{\text{fabrication data}} \rightarrow \underbrace{\text{ITF (.ICT)}}_{\text{text, per corner}} \xrightarrow{\text{Techgen}} \underbrace{\text{qrcTechFile (binary)}}_{\text{per RC corner}} \rightarrow \begin{cases} \text{Innovus (P\&R)} \rightarrow \text{SPEF (estimated)} \\ \text{QRC (RC Extraction)} \rightarrow \text{SPEF (accurate)} \end{cases}$$

**Tại sao có hai paths song song?** Innovus dùng qrcTechFile để estimate RC trong quá trình PnR optimization (trước khi có full routing geometry). QRC dùng qrcTechFile để extract RC chính xác từ actual wire geometries sau Routing. Cả hai produce SPEF nhưng accuracy khác nhau: Innovus SPEF là approximation, QRC SPEF là ground truth cho Signoff.

## Computed from

Các thông số R/C trong ITF được derive từ material properties và layer geometry:

**Sheet Resistance** (resistance per square):
$$R_\square = \frac{\rho}{H}$$

Trong đó $\rho$ = resistivity của metal material (e.g., Cu), $H$ = thickness của metal layer. Wire resistance sau đó:
$$R_\text{wire} = R_\square \times \frac{L}{W}$$

**Parallel Plate Capacitance** (dielectric giữa metal và ground plane):
$$C_\text{pp} = \frac{\varepsilon}{t_\text{di}} \times W \times L$$

**Total Ground Capacitance** (bao gồm fringe effects):
$$C_\text{ground} = C_\text{pp} + 2 \times C_\text{fringe}$$

Mỗi metal layer có $R_\square$ và $C$ riêng — upper layers (thick, wide Pitch) có $R_\square$ thấp hơn và phù hợp cho global routing; lower layers (thin, fine Pitch) có $R_\square$ cao hơn và phù hợp cho local routing.

RC corner naming convention phản ánh worst-case combinations:

| RC Corner | Process Variation | Temperature | Net Delay impact |
|---|---|---|---|
| rcworst | Max R + Max C | High (125°C) | Max → worst Setup |
| rcbest | Min R + Min C | Low (−40°C) | Min → worst Hold |
| typical | Nominal | Nominal (25°C) | Reference |

## Constrains
- **[[NetDelay]]**: $R$ và $C$ values từ ITF (qua SPEF) là input trực tiếp cho timing của routed interconnect; accuracy của [[InterconnectRC]] trong SPEF phụ thuộc vào accuracy của technology data
- **[[MMMC]]**: mỗi RC Corner trong MMMC file tham chiếu một qrcTechFile theo corner name; incorrect qrcTechFile pairing → incorrect Net Delay → incorrect Slack → false timing closure

## Requires
- [[MetalStack]] — layer structure (layer type, thickness, dielectric material, spacing) mà ITF characterize

## Used by
- [[MMMC]] — RC Corner definitions trong MMMC embed qrcTechFile reference; ITF load gián tiếp khi MMMC file được read
- [[ParasiticExtraction]] — QRC tool dùng qrcTechFile để extract actual R/C từ wire geometries sau Routing → output SPEF cho Signoff STA
- [[DesignImport]] — qrcTechFile được reference trong MMMC, load cùng với MMMC setup

## Key insight
[USER REVIEW — draft suggestion]: ITF là "bridge" giữa Foundry physics và EDA tool math. Foundry đo điện trở và điện dung thực trên wafer (dùng test structures trên PCM), fit thành R/C tables trong ITF. EDA tool dùng những tables này để predict Net Delay trước khi chip được fab. Accuracy của ITF trực tiếp ảnh hưởng đến correlation giữa PnR simulation và silicon measurement — poor ITF → timing closure ở tool nhưng fail trên silicon. Đây là lý do Foundry PDK updates (thường đi kèm refined ITF) yêu cầu re-signoff toàn bộ design.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Source: Foundry PDK
→ Pipeline: ITF (.ICT) → Techgen → qrcTechFile (binary, per RC corner)
→ Two consumers: Innovus (estimated SPEF) · QRC (accurate SPEF for Signoff)
→ Constrains: [[InterconnectRC]] · [[NetDelay]] (via R/C values in SPEF)
→ Part of: [[MMMC]] RC Corner definitions
→ Cùng nhóm: [[MetalStack]] · [[SPEF]] · [[ParasiticExtraction]]