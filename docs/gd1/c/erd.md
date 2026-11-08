# ERD Module C

Bản hiện hành sau GD1-03. Sơ đồ PNG trước đây chưa có liên kết gói đã mua; dùng sơ đồ này khi ghép ERD tổng. Thực thể A/B chỉ hiện cột cần tham chiếu trong hợp đồng; quy tắc ngoài sơ đồ ở [business-rules.md](business-rules.md).

```mermaid
erDiagram
    ChiNhanh ||--|{ CuaKiemSoat : co
    ChiNhanh ||..o{ CheckIn : phat_sinh
    CuaKiemSoat ||..o{ CheckIn : cua_vao
    CuaKiemSoat |o..o{ CheckIn : cua_ra
    HoiVien ||..o{ CheckIn : thuc_hien
    HoiVien ||..o{ HoaDon : mua
    HoaDon ||--|{ ChiTietHoaDon : gom
    GoiDichVu |o..o{ ChiTietHoaDon : dong_goi
    ChiTietHoaDon ||..o{ GoiHoiVien : tao_don_vi
    GoiHoiVien ||..o{ LichHenPT : cap_buoi
    NhanVien ||..o{ LichHenPT : huan_luyen
    ChiNhanh ||..o{ LichHenPT : noi_tap
    LichHenPT ||..o| DanhGiaHLV : duoc_danh_gia
    ChiNhanh {
        string MaCN PK
        int SucChua
        time GioMoCua
        time GioDongCua
    }
    NhanVien {
        string MaNV PK
        string ChucVu
    }
    HoiVien {
        string MaHV PK
    }
    GoiDichVu {
        string MaGoi PK
        string LoaiGoi
    }
    HoaDon {
        string MaHD PK
        string MaHV FK
    }
    ChiTietHoaDon {
        string MaHD PK, FK
        int STT PK
        string MaGoi FK
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
    CuaKiemSoat {
        string MaCN PK, FK
        int SoHieuCua PK
        string ChucNang
        boolean DangHoatDong
    }
    CheckIn {
        string MaCheckIn PK
        string MaHV FK
        string MaCN FK
        int SoHieuCuaVao FK
        int SoHieuCuaRa FK
        datetime TGVao
        datetime TGRa
    }
    LichHenPT {
        string MaLichHen PK
        string MaGHV FK
        string MaNV FK
        string MaCN FK
        datetime NgayGio
        datetime ThoiDiemDat
        string TrangThai
        boolean DaTruBuoi
        string LyDoKetThuc
    }
    DanhGiaHLV {
        string MaDanhGia PK
        string MaLichHen FK, UK
        int Diem
        string NhanXet
        datetime ThoiDiemDanhGia
    }
```

- Hai FK cửa của CheckIn là cặp `(MaCN, SoHieuCuaVao)` và `(MaCN, SoHieuCuaRa)`; cửa ra có thể rỗng khi đang mở hoặc tự đóng.
- MaGHV bắt buộc, chỉ gắn gói loại PT. MaHV của lịch suy ra từ gói, không lưu trùng; gói có thể không có lịch hoặc có nhiều lịch, kể cả lịch đã hủy.
- Quan hệ hội viên–lịch được thể hiện qua gói đã mua. Mỗi lịch thuộc đúng một hội viên. Tra cứu lịch sử và kiểm tra chủ đánh giá phải nối qua GoiHoiVien.
- Số giữ chỗ, chọn gói đủ hạn, trừ đúng một lần và yêu cầu cập nhật nhất quán được mô tả trong [thiết kế PT B–C](../b/goi-pt.md).

GD1-11: MaHV/MaGoi đọc qua vGoiHoiVien từ hóa đơn/dòng; FK lịch vẫn tới MaGHV bảng cơ sở. FK dòng là (MaHD,STT); AK đơn vị là (MaHD,STT,DonVi).
