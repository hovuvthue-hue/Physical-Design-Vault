# Kiến trúc:

Vault/
├── 00_MOC/                     ← Map of Content — điều hướng
│   ├── Physical Design Flow.md
│   ├── LEF Geometry Group.md
│   └── STA Concepts Group.md
│
├── 01_Chains/                  ← Dependency Chain — quan hệ nhân quả
│   ├── Chain_Pitch_to_RoutingGrid.md
│   ├── Chain_Site_to_Placement.md
│   └── Chain_LEF_to_PnR.md
│
├── 02_Concepts/                ← Concept Card — 1 file = 1 khái niệm
│   ├── Pitch.md
│   ├── Track.md
│   ├── Site.md
│   ├── Row.md
│   ├── LEF.md
│   └── ...
│
└── 03_Reference/               ← Bảng tổng hợp, file format matrix
    ├── File_Format_Exchange.md
    └── PPARMP_Tradeoffs.md

# Schema Concept Card trong Obsidian

Mỗi file trong `02_Concepts/` dùng **YAML frontmatter** cho metadata máy đọc được, và **body cố định** cho nội dung con người đọc:

File trong `01_Chains/` là nơi **duy nhất** thể hiện quan hệ có hướng và có nhãn:
