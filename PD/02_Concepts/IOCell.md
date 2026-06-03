---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: IO Cell Library — provided by External IP Vendor; placed in IO Ring during Floorplanning
used_by: [Floorplanning, PhysicalConstraints, CoreArea]
requires: [LEF, LIB, GDS]
chain: Chain_PnR_Flow
---
# IOCell

## Definition
IO Cell là các Standard Cells đặc biệt tạo thành IO Ring — vành đai ngoại vi của chip kết nối thế giới bên trong (Core logic) với thế giới bên ngoài (Package/PCB). IO Cells có kích thước vật lý lớn hơn nhiều so với Standard Cells thông thường và **không scale theo Moore's law** — dù transistors bên trong lõi thu nhỏ xuống 3–5nm, IO Cells vẫn phải đủ lớn để chịu được điện dung/điện cảm của PCB traces và để accommodate wire bonding/flip-chip bumps.

**Taxonomy IO Cells:**

| Loại | Chức năng | Đặc điểm |
|---|---|---|
| Digital I/O Buffer | Level shifting + ESD protection cho digital signals | Input Buffers, Output Drivers, Bi-directional; phải drive pF load (vs fF bên trong Core) |
| Analog I/O Cell | Truyền analog signals vào/ra chip | Thực chất là "wire" với ESD protection; không có logic |
| Power Supply Cell | Cung cấp VDD/VSS cho IO Ring và Core | Analog cells; đóng vai trò vành đai phân phối nguồn |
| Corner Cell | Đặt tại 4 góc của IO Ring | 45° wires bên trong để minimize current crowding, improve EM |
| Filler Cell (IO Filler) | Lấp khoảng trống trong IO Ring | Đảm bảo VDD/VSS rings liên tục; electrical connection |

**Digital I/O Buffer — ràng buộc điện đặc thù:**

Digital I/O Buffer khác hoàn toàn với Standard Cell Buffer thông thường về yêu cầu
drive strength:

- **Load target: pF, không phải fF** — IO buffer phải drive PCB trace, bond wire,
  và package capacitance (~pF range), so với on-chip load chỉ ở fF range.
  Đây là lý do IO cell phải có transistor width lớn hơn nhiều so với logic cell.

- **Requires fanout inverter chain** — để tạo được drive strength đủ lớn,
  IO output buffer được implement như một chuỗi inverters với kích thước tăng dần
  (tapered inverter chain) để drive large external load mà không tạo
  excessively large input capacitance tại stage đầu.

- **Short-circuit current là unacceptable** — trong quá trình switching,
  PMOS và NMOS cùng dẫn trong một khoảng thời gian ngắn tạo short-circuit current.
  Ở on-chip speed với fF load, thời gian này cực ngắn và ảnh hưởng không đáng kể.
  Nhưng với pF external load, switching chậm hơn → overlap time dài hơn →
  short-circuit current tăng đáng kể → phải được kiểm soát trong IO buffer design.
  Đây là một trong các lý do IO cell characterization phức tạp hơn Standard Cell.

**Corner Cell — 45-degree wires:**

Corner Cell được đặt tại 4 góc của IO Ring. Điểm đặc trưng là sử dụng 45-degree wires
thay vì right-angle routing tại điểm chuyển hướng. Bốn lý do kỹ thuật:

1. **Minimize Current Crowding**: right-angle corners tạo điểm hội tụ dòng điện cục bộ
   (current crowding) làm tăng local current density.
2. **Improve EM Performance**: current crowding trực tiếp làm tăng rủi ro Electromigration
   tại corner. 45° spread out dòng điện đều hơn.
3. **Enhance Signal Integrity**: giảm impedance discontinuity tại điểm chuyển hướng,
   cải thiện signal quality trên high-frequency IO.
4. **Reduce Metal Density**: phân bố metal geometry đều hơn tại corner,
   hỗ trợ CMP uniformity và metal density DRC compliance.

**Tại sao IO power supply tách biệt với Core power:**

IO Cells tiêu thụ dòng điện lớn và thường chạy ở điện áp cao hơn Core (ví dụ: IOVDD = 2.5V vs VDD = 1.2V). Nếu dùng chung power supply, switching noise từ IO sẽ couple vào Core logic → setup/hold violations. Core và IO power domains phải tách biệt với separate power rings.

**Electrical Interface Standards:**

Các IO Cells đặc biệt (Special IO) tuân theo interface standards cụ thể: SSTL (Stub Series Terminated Logic — DRAM interfaces), HSTL (High Speed Transceiver Logic — high-speed memories), LVDS (Low Voltage Differential Signaling — high-bandwidth links). Mỗi standard quy định voltage levels, termination requirements, và timing budgets riêng.

**ESD Protection:**

Electrostatic Discharge (ESD) là một trong những reliability problems quan trọng nhất trong IC industry. Cơ chế protection:
- Primary ESD Elements: Diode clamps kích hoạt khi voltage vượt VDD + 0.7V hoặc giảm dưới VSS − 0.7V, divert high current away from internal circuitry
- Current Limiting Resistor: giới hạn current để bảo vệ secondary ESD elements
- Secondary ESD Elements: bảo vệ thin gate oxides của internal circuits

**Seal Ring và Scribe Line:**

Seal Ring là cấu trúc bảo vệ bao quanh toàn bộ active die area, nằm giữa IO Ring và Scribe Line. Cấu tạo từ wide metal layers stacking (M1 → Mtop) cùng với diffusion, well, contacts/vias. Chức năng: (1) chống mechanical damage khi wafer dicing, (2) ngăn moisture/contamination ingress, (3) chống electrical overstress từ external fields.

Scribe Line (saw street) là vùng non-functional giữa các dies trên wafer, nơi wafer saw sẽ cắt. Phải giữ Macros và IO Cells đủ xa die edge để không vi phạm Seal Ring và không bị ảnh hưởng bởi mechanical stress từ dicing.

**Lưu ý Seal Ring với Continuous Ring:** Continuous Seal Ring tạo ra conductor loop → unwanted signal propagation (noise propagates along ring, interferes với sensitive circuits như PLL, analog blocks). Solution: sử dụng Seal Ring với gaps (segmented) để ngăn noise propagation.

## Computed from

**Phân biệt full-chip và block-level:**
- **Full-chip:** đặt IO Pad để giao tiếp với Package/PCB, kèm ràng buộc ESD/power/packaging.
- **Block-level:** IO Pin là interface nội bộ giữa các blocks, thường được inherit từ top-level floorplan và phải honor đúng vị trí đã assign.

**IO Pad Placement strategy:**

Chip-level design: Engineer xác định vị trí từng IO Pad trên 4 cạnh chip. Thứ tự sắp xếp phải được thống nhất với Package team để:
- Wire bonds không cross nhau (No Crossing rule)
- Wire bond length trong Maximum Length limit
- Wire bond angle trong Maximum Angle limit
- Minimum Spacing giữa adjacent bond pads được đảm bảo

Block-level design: IO Pin locations được kế thừa từ top-level floorplan — block designer không tự quyết định, phải honor exactly vị trí đã được top-level integration team assign. Block Pins thường được đặt trên M3 (metal layer đủ cao để Router có thể access từ cả hai phía boundary) và được placed dựa trên Timing (minimize distance đến related registers) và Congestion (distribute Pins đều trên boundary, tránh cluster).

## Constrains
- **[[CoreArea]]**: IO Ring Area chiếm phần ngoài của Die — IO Cell height × số lượng Pads quyết định Core-to-IO Spacing requirement
- **[[PhysicalConstraints]]**: IO Pad instance positions trong `.io` file phải match GateLevelNetlist instance names
- **[[Routing]]**: Signals từ Core phải route từ Core Pins đến IO Cell Pins — routing từ Core boundary ra IO Ring thường đi qua dedicated routing channels

## Requires
- [[LEF]] — IO Cell CellAbstract (SIZE, Pins, OBS)
- [[LIB]] — IO Cell timing model (drive strength, setup/hold times tại IO boundary)
- [[GDS]] — Full IO Cell layout cho Tape-out

## Used by
- [[Floorplanning]] — IO Cells placed trong sub-step "Place IOs" của Floorplanning
- [[PhysicalConstraints]] — IO Pad placement file (.io) reference IO Cell instance names
- [[CoreArea]] — IO Ring Area là một trong 3 thành phần của Die Area formula

## Key insight
[USER REVIEW — draft suggestion]: IO Cells thường là nguyên nhân gây ra "Pad-limited design" — một vấn đề thường gặp trong chips có high I/O count (networking chips, memory controllers). Giải pháp Circuit Under Pad (CUP) cho phép đặt bonding pad trực tiếp trên IO body (thay vì dùng diện tích riêng), tiết kiệm die area đáng kể. Tuy nhiên CUP tăng độ phức tạp của IO cell characterization và đòi hỏi special layout rules. Flip Chip với RDL là evolution tiếp theo — IO bumps phân bố trên toàn bề mặt chip thay vì bị giới hạn ở periphery, giải quyết triệt để vấn đề Pad-limited.

## Related
→ Chain: [[Chain_PnR_Flow]]
→ Types: Digital I/O · Analog I/O · Power Supply · Corner Cell · Filler Cell
→ Special: ESD Protection · Seal Ring · Scribe Line
→ Configurations: Inline · Staggered Non-CUP · Staggered CUP · Flip Chip with RDL
→ Interface Standards: SSTL · HSTL · LVDS
→ Constrains: [[CoreArea]] · [[PhysicalConstraints]] · [[Routing]]
→ Cùng nhóm: [[PhysicalConstraints]] · [[Floorplanning]] · [[CoreArea]]