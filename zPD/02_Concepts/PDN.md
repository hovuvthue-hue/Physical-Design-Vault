---
tags: [concept, pnr-flow]
group: PnR Flow
defined_in: Floorplanning
used_by: [Floorplanning, Routing, Signoff]
requires: [Floorplanning, StandardCell, HardIP, MetalStack]
chain: Chain_PnR_Flow
---
# PDN

## Definition
PDN (Power Distribution Network) là mạng phân phối nguồn điện trong layout vật lý, có nhiệm vụ đưa nguồn VDD/VSS từ vùng cấp nguồn ở biên chip vào toàn bộ vùng logic bên trong core. Trong bối cảnh [[Floorplanning]], PDN là phần cốt lõi của Power Planning, vì nếu cấu trúc nguồn yếu thì các bước sau như [[Routing]] và [[Signoff]] sẽ gặp rủi ro lớn về power integrity.

## Mục tiêu của Power Planning
Một PDN tốt cần cân bằng đồng thời các mục tiêu sau:
- Cấp nguồn đến tất cả cells và macros một cách liên tục.
- Giảm rủi ro [[IRDrop]] để hạn chế sụt áp cục bộ.
- Giảm rủi ro Electromigration / self-heating ở mức khái niệm (không đặt ngưỡng số).
- Tránh over-design: lưới nguồn quá dày có thể tiêu tốn diện tích kim loại và làm giảm Routing Resource cho tín hiệu.

## Cấu trúc phân cấp của PDN
PDN thường được tổ chức theo phân cấp từ ngoài vào trong:
1. **PG Pads / IO Ring**: điểm nhận nguồn từ package vào chip.
2. **Core Ring**: vòng nguồn bao quanh core để phân phối dòng vào vùng logic.
3. **Power Stripes / Power Straps**: các tuyến nguồn chính đi xuyên core để đưa nguồn sâu vào bên trong.
4. **Macro / Block Rings**: vòng nguồn cục bộ cho các macros/blocks lớn.
5. **Standard Cell PG Rails (Standard Cell Rail)**: thanh nguồn cục bộ trong hàng standard cell.
6. **VDD/VSS pins**: chân nguồn của từng cell/macro, là điểm tiêu thụ cuối.

## Multi-layer distribution (concept level)
PDN thường dùng nhiều lớp kim loại với vai trò khác nhau:
- Lớp kim loại cao: thiên về phân phối nguồn diện rộng, điện trở thấp hơn.
- Lớp trung gian: làm tầng trung chuyển giữa ring và rails.
- Lớp thấp: cấp nguồn cục bộ đến Standard Cell Rails và pins.

Mapping lớp cụ thể (ví dụ Mx nào cho ring/strap/rail) là phụ thuộc process, PDK và rule deck. **[Needs verification]**

## Power/Ground Nets trong physical design
Trong luồng thực tế, netlist logic tổng hợp vào PnR có thể chưa thể hiện đầy đủ kết nối nguồn/ground theo topology vật lý. Vì vậy, trong giai đoạn Floorplanning/PDN setup, kỹ sư cần thiết lập và bảo đảm kết nối đúng cho:
- VDD/VSS nets của standard cells,
- PG pins của macros,
- các phần tử liên quan nguồn/ground như tie connectivity theo flow.

Ý nghĩa: PowerNets không chỉ là “tên net” trong logic, mà là một cấu trúc kết nối vật lý cần được xây dựng đầy đủ trong PDN.

## Trade-off chính
PDN càng robust thì rủi ro [[IRDrop]] càng giảm, nhưng thường chiếm thêm tài nguyên định tuyến. Do đó Power Planning là bài toán cân bằng giữa độ bền nguồn và khả năng đi dây tín hiệu, thay vì tối ưu một phía tuyệt đối.

## Related
- Upstream: [[Floorplanning]] · [[StandardCell]] · [[HardIP]]
- Downstream: [[Routing]] · [[Signoff]] · [[IRDrop]]
- Cùng nhóm: [[PhysicalConstraints]] · [[MetalStack]]
