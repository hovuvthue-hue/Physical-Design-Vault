---
tags: [concept, cell-library, pnr-flow]
group: Cell Library
defined_in: Standard Cell Library; inserted during floorplanning/placement preparation
used_by: [Floorplanning, Placement, PDN, Signoff]
requires: [StandardCell, Row, PDN]
chain: Chain_LEF_to_PnR
---
# TapCell

## Definition
TapCell là **physical-only standard cell** dùng để nối các vùng well/substrate (body regions) về các bias rails phù hợp (ví dụ Well Tap / Substrate Tap) trong layout. TapCell không mang logic function xử lý dữ liệu; vai trò chính là đảm bảo điều kiện bias vật lý đúng cho nền công nghệ.

## Why TapCell exists
Ở mức khái niệm, TapCell giúp:
- duy trì well/substrate bias integrity,
- hỗ trợ giảm rủi ro **Latch-up**,
- ổn định điều kiện vận hành vật lý của Standard Cell fabric.

**Lịch sử và lý do tồn tại như cell riêng:**

Trong các Standard Cell library thế hệ đầu (**Traditional SC**), mỗi cell đã tích hợp sẵn
kết nối N-Well → VDD và P-Substrate → VSS bên trong layout của chính nó.
Cách thiết kế này đảm bảo latch-up prevention nhưng tốn diện tích cell.

Để tối ưu cell area, các library hiện đại chuyển sang kiến trúc **Tapless SC** — loại bỏ
hoàn toàn internal well tap connections, nhờ đó giảm được cell footprint. Đánh đổi là
must insert standalone Tap Cells định kỳ vào design để đảm bảo mọi vùng
N-Well và P-Substrate đều được bias đúng. Tap Cell vì vậy là thành phần bắt buộc của
Tapless Standard Cell Library.

Ba kiến trúc liên quan (xem [[StandardCell]] — p.70 L5):
- **Traditional SC**: internal tap tích hợp sẵn, không cần Tap Cell riêng.
- **Tapless SC**: không có internal tap, phụ thuộc Tap Cell để đảm bảo well/substrate bias.
- **Tap Cell**: physical-only cell chuyên dụng cho well/substrate connection — **không có input/output logic pins**.

TapCell vì vậy thuộc nhóm cell bắt buộc cho reliability trong triển khai PnR, ngay cả khi không tham gia logic computation.

## Relation to rows / wells / substrate
TapCell được chèn theo cấu trúc [[Row]] và vùng placement của standard cells để bảo đảm mỗi khu vực well/substrate được “neo” về rail bias phù hợp trong khoảng phủ hợp lệ.

Kiểu triển khai Well Tap/Substrate Tap cụ thể phụ thuộc architecture của library và quy tắc process. **[Needs verification]**

## PDK dependence
Rule về mật độ chèn, khoảng cách, topology tap network và sequencing trong flow là **PDK/foundry/tool dependent**. Không thể dùng một giá trị spacing chung cho mọi node/process.

Bất kỳ con số insertion interval cụ thể nào đều phải lấy từ deck/flow guideline tương ứng. **[Needs verification]**

## Constrains
- TapCell insertion phải tương thích với row legality và power rail orientation.
- Không trộn vai trò TapCell với TieCell: TapCell cho body-bias, TieCell cho logic constant.
- Rule chi tiết về placement density/interval là process-specific. **[Needs verification]**

## Requires
- [[StandardCell]] — TapCell là thành phần thuộc cell library.
- [[Row]] — insertion bám theo cấu trúc Standard Cell Row.
- [[PDN]] — cần power/ground bias context để ràng buộc well/substrate tie.

## Used by
- [[Floorplanning]] — xác lập chiến lược physical-only cell ở giai đoạn chuẩn bị layout.
- [[Placement]] — thực thi/legalize insertion theo row topology.
- [[PDN]] — liên quan đến bias rail connectivity mức khái niệm.
- [[Signoff]] — kiểm tra các điều kiện reliability và tính đầy đủ kết nối.

## Related
→ Chain: [[Chain_LEF_to_PnR]]
→ Phân biệt với: [[TieCell]]
→ Cùng nhóm: [[StandardCell]] · [[Row]] · [[PDN]] · [[EndCapCell]]
