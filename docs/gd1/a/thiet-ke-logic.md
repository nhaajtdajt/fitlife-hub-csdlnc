# Thiết kế logic — Module A: Chi nhánh, nhân sự và hoa hồng

Thiết kế logic đầy đủ của toàn hệ thống (lược đồ, khóa, phụ thuộc hàm, dạng chuẩn, mô tả thuộc tính, khóa ngoại, ràng buộc) nằm ở **[thiết kế logic tổng](../thiet-ke-tong.md)**, hình vẽ tại [luoc-do-logic.png](../luoc-do-logic.png). File này chỉ tóm phần của Module A để tiện tra.

## Quan hệ do Module A sở hữu

- CHI_NHANH
- NHAN_VIEN
- LICH_SU_CONG_TAC
- HOA_HONG_CHI_NHANH

Khóa, phụ thuộc hàm, mô tả thuộc tính: xem mục 2 và mục 4 của [thiết kế tổng](../thiet-ke-tong.md).

## Dạng chuẩn

Cả bốn quan hệ đạt BCNF.

## Ràng buộc liên quan

R01–R03 (miền), R07–R09; A còn chịu ảnh hưởng của R18, R25 khi sửa lịch sử công tác — mục 6 của [thiết kế tổng](../thiet-ke-tong.md).

## Dữ liệu dùng từ module khác

`LICH_HEN_PT` (C) để tính hoa hồng và kiểm tra lịch vướng.
