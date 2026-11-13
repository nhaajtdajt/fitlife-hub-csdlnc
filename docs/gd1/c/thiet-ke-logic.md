# Thiết kế logic — Module C: Check-in, lịch hẹn PT và đánh giá HLV

Thiết kế logic đầy đủ của toàn hệ thống (lược đồ, khóa, phụ thuộc hàm, dạng chuẩn, mô tả thuộc tính, khóa ngoại, ràng buộc) nằm ở **[thiết kế logic tổng](../thiet-ke-tong.md)**, hình vẽ tại [luoc-do-logic.png](../luoc-do-logic.png). File này chỉ tóm phần của Module C để tiện tra.

## Quan hệ do Module C sở hữu

- CHECK_IN
- LICH_HEN_PT
- DANH_GIA_HLV

Khóa, phụ thuộc hàm, mô tả thuộc tính: xem mục 2 và mục 4 của [thiết kế tổng](../thiet-ke-tong.md).

## Dạng chuẩn

CHECK_IN, DANH_GIA_HLV đạt BCNF; LICH_HEN_PT đạt 2NF do cố ý lưu `MaCN` (`{MaNV, NgayTap} → MaCN`).

## Ràng buộc liên quan

R06 (miền), R14–R22 — mục 6 của [thiết kế tổng](../thiet-ke-tong.md).

## Dữ liệu dùng từ module khác

`GOI_HOI_VIEN`, `HANG_MUC_CUNG_CAP` (B); `CHI_NHANH`, `LICH_SU_CONG_TAC` (A).
