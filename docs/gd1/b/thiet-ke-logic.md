# Thiết kế logic — Module B: Hội viên, gói dịch vụ và điểm/hạng

Thiết kế logic đầy đủ của toàn hệ thống (lược đồ, khóa, phụ thuộc hàm, dạng chuẩn, mô tả thuộc tính, khóa ngoại, ràng buộc) nằm ở **[thiết kế logic tổng](../thiet-ke-tong.md)**, hình vẽ tại [luoc-do-logic.png](../luoc-do-logic.png). File này chỉ tóm phần của Module B để tiện tra.

## Quan hệ do Module B sở hữu

- HOI_VIEN
- HANG_MUC_CUNG_CAP (các dòng loại gói; dùng chung với D)
- GOI_HOI_VIEN

Khóa, phụ thuộc hàm, mô tả thuộc tính: xem mục 2 và mục 4 của [thiết kế tổng](../thiet-ke-tong.md).

## Dạng chuẩn

HANG_MUC_CUNG_CAP, GOI_HOI_VIEN đạt BCNF; HOI_VIEN đạt 2NF do cố ý lưu `Hang` (`TongChiTieu → Hang`).

## Ràng buộc liên quan

R04–R05 (miền), R10–R13 — mục 6 của [thiết kế tổng](../thiet-ke-tong.md).

## Dữ liệu dùng từ module khác

`HOA_DON`, `CHI_TIET_HOA_DON` (D) khi tạo gói và cộng điểm; `CHECK_IN`, `LICH_HEN_PT` (C) giảm `SoBuoiConLai`.
