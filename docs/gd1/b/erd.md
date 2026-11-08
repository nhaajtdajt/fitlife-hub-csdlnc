# ERD Module B

Bản hiện hành GD1-11, đồng bộ [thiết kế logic](thiet-ke-logic.md). Chỉ hiện phần dữ liệu A/D cần tham chiếu.

```mermaid
erDiagram
    ChiNhanh ||..o{ HoiVien : dang_ky
    HoiVien ||..o{ HoaDon : mua
    HoaDon ||--|{ ChiTietHoaDon : gom
    GoiDichVu |o..o{ ChiTietHoaDon : dong_goi
    ChiTietHoaDon ||..o{ GoiHoiVien : tao_don_vi
    HoaDon ||..o| GhiNhanLoyalty : ghi_nhan
    HoiVien {
        string MaHV PK
        string HoTen
        string SDT
        string Email
        string CCCD
        string GioiTinh
        date NgaySinh
        date NgayDangKy
        string MaCNDangKy FK
        decimal TongChiTieu12T
        int DiemTichLuy
        string Hang
    }
    GoiDichVu {
        string MaGoi PK
        string TenGoi
        string LoaiGoi
        string CachTinh
        int ThoiHanThang
        int SoBuoiBanDau
        decimal Gia
        boolean DangKinhDoanh
    }
    GoiHoiVien {
        string MaGHV PK
        string MaHD FK
        int STT FK
        int DonVi
        date NgayBatDau
        date NgayHetHan
        int SoBuoiConLai
    }
    GhiNhanLoyalty {
        string MaHD PK, FK
        int DiemCong
        datetime ThoiDiemGhiNhan
        boolean DaLoaiKhoi12T
    }
    HoaDon {
        string MaHD PK
        string MaHV FK
        datetime NgayLap
        decimal TongTien
    }
    ChiTietHoaDon {
        string MaHD PK, FK
        int STT PK
        string LoaiDong
        string MaGoi FK
        int SoLuong
    }
    ChiNhanh {
        string MaCN PK
    }
```

GoiHoiVien có FK ghép (MaHD,STT), AK (MaHD,STT,DonVi). SDT/CCCD của HoiVien là AK. Một dòng gói đã thanh toán tạo đúng SoLuong gói; dòng sản phẩm tạo 0. Hóa đơn đã hoàn tất có đúng một ghi nhận Loyalty. Hội viên và mã danh mục của gói đã mua suy ra qua hóa đơn/dòng; vGoiHoiVien là view đọc, không phải thực thể lưu thêm. Ràng buộc thời gian/điểm/hạng ở thiết kế logic.
