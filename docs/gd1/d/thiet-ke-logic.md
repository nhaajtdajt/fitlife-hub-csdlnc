# Thiết kế logic — Module D: Sản phẩm, tồn kho, hóa đơn và báo cáo

Thiết kế logic đầy đủ của toàn hệ thống (lược đồ, khóa, phụ thuộc hàm, dạng chuẩn, mô tả thuộc tính, khóa ngoại, ràng buộc) nằm ở **[thiết kế logic tổng](../thiet-ke-tong.md)**, hình vẽ tại [luoc-do-logic.png](../luoc-do-logic.png). File này chỉ tóm phần của Module D để tiện tra.

## Quan hệ do Module D sở hữu

- HANG_MUC_CUNG_CAP (các dòng loại Sản phẩm; dùng chung với B)
- TON_KHO
- HOA_DON
- CHI_TIET_HOA_DON

Khóa, phụ thuộc hàm, mô tả thuộc tính: xem mục 2 và mục 4 của [thiết kế tổng](../thiet-ke-tong.md).

## Dạng chuẩn

TON_KHO, CHI_TIET_HOA_DON đạt BCNF; HOA_DON đạt 2NF do cố ý lưu `MaCN` (`{MaNV, NgayLap} → MaCN`).

## Ràng buộc liên quan

R06 (miền), R23–R27 — mục 6 của [thiết kế tổng](../thiet-ke-tong.md).

## Dữ liệu dùng từ module khác

`HOI_VIEN` (B); `NHAN_VIEN`, `CHI_NHANH` (A); `CHECK_IN` (C) cho báo cáo lấp đầy.
