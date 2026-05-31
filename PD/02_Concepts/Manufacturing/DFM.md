---
tags: [concept, manufacturing, signoff, physical-verification]
group: Manufacturing
defined_in: Manufacturing / Signoff
used_by: [ChipFinishing, Signoff]
requires: [DRC, Routing, ChipFinishing]
chain: Chain_PnR_Flow
---
# DFM

## Definition
DFM (Design for Manufacturability) là nhóm nguyên tắc thiết kế/layout và kiểm tra nhằm làm design dễ chế tạo hơn và robust hơn trước các biến thiên manufacturing. Trong Physical Design, DFM nhìn vào routed layout không chỉ như connectivity đúng/sai, mà còn như hình học sẽ được in, etch, deposit và kiểm tra trong process thật.

DFM không bảo đảm yield. Nó giảm rủi ro manufacturability bằng cách làm layout phù hợp hơn với rule deck, foundry guidance và signoff methodology; chi tiết cụ thể phụ thuộc foundry, PDK, process node và policy nội bộ. [Needs verification]

## DFM vs DRC
[[DRC]] kiểm tra layout có vi phạm các rule hình học bắt buộc hay không. Một design DRC-clean vẫn có thể chưa tối ưu về DFM nếu còn các pattern, density, via robustness hoặc manufacturing-variability concerns mà methodology muốn cải thiện thêm. [Needs verification]

Vì vậy DRC là gate rule-compliance cơ bản, còn DFM là góc nhìn manufacturability-aware rộng hơn. DFM có thể bao gồm các check hoặc optimization vượt ngoài câu hỏi “có vi phạm rule cứng không?”, nhưng mọi tiêu chí cụ thể phải theo rule deck/foundry/signoff policy. [Needs verification]

## Relation to ChipFinishing
[[ChipFinishing]] là vùng thực thi muộn giúp áp dụng một phần DFM trên layout đã route. Ví dụ concept-level gồm fill/density balancing, via robustness, cleanup antenna handoff và final physical consistency trước [[Signoff]].

Nếu ChipFinishing thay đổi geometry, kết quả timing, extraction hoặc physical verification có thể cần cập nhật lại tùy flow. [Needs verification]

## Common DFM concerns
Các chủ đề DFM thường gặp trong routing/signoff context gồm:

- metal density và fill để hỗ trợ process uniformity;
- via robustness, trong đó RedundantVia hoặc multi-cut via có thể được dùng khi hợp lệ;
- lithography/manufacturing-variability awareness đối với pattern khó chế tạo;
- [[AntennaEffect]] như rủi ro manufacturing liên quan route/fabrication;
- tương quan giữa in-tool checks và signoff/foundry rule decks. [Needs verification]

Các mục này là concept-level. Card này không định nghĩa rule values, density thresholds, command, report hoặc foundry checklist.

## Limits / foundry dependence
- DFM criteria phụ thuộc foundry/PDK/process node/rule deck/signoff policy. [Needs verification]
- DFM không thay thế [[DRC]] hoặc [[Signoff]].
- DFM không đồng nghĩa với guarantee yield.
- Card này không tạo concept riêng cho MetalFill, RedundantVia, ERC hoặc Electromigration.

## Requires
- [[DRC]] — cung cấp nền rule-compliance và physical verification context.
- [[Routing]] — tạo wire/via geometry mà DFM sẽ đánh giá hoặc cải thiện.
- [[ChipFinishing]] — vùng finishing muộn nơi nhiều hành động DFM có thể được áp dụng.

## Used by
- [[ChipFinishing]] — dùng DFM như mục tiêu manufacturability-aware cho các hoạt động hoàn thiện.
- [[Signoff]] — đặt DFM trong bối cảnh final qualification và release readiness.

## Related
→ Physical verification: [[DRC]] · [[Signoff]]
→ Flow: [[Routing]] · [[ChipFinishing]]
→ Manufacturing risk: [[AntennaEffect]]
