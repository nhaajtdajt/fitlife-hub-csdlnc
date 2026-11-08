# ERD A hiện hành sau GD1-10

Sơ đồ quan hệ hiện hành dùng cùng [thiết kế logic](thiet-ke-logic.md). erd-a.drawio giữ bản quan niệm trước phần chốt kỳ; phân cấp nhân viên ánh xạ bằng ChucVu, quan hệ cung cấp có thời gian ánh xạ CungCapTienIch.

```mermaid
erDiagram
    ChiNhanh ||--o{ CungCapTienIch : cung_cap
    TienIch ||--o{ CungCapTienIch : duoc_cung_cap
    NhanVien ||--|{ LichSuCongTac : trai_qua
    ChiNhanh ||--o{ LichSuCongTac : noi_cong_tac
    ChiNhanh ||--o{ KyHoaHong : chot
    NhanVien ||..o{ KyHoaHong : nguoi_chot
    KyHoaHong ||--o{ ChiTietHoaHong : gom
    LichHenPT ||..o| ChiTietHoaHong : duoc_tinh
    NhanVien ||..o{ LichHenPT : day
    ChiNhanh ||..o{ LichHenPT : noi_day
    ChiNhanh {
        string MaCN PK
        string TenCN
        string DiaChi
        string SDT
        time GioMoCua
        time GioDongCua
        int SucChua
        decimal HoaHongMoiBuoi
        string TrangThai
    }
    TienIch {
        string MaTI PK
        string TenTI
        string Loai
        string MoTa
        string TrangThai
    }
    CungCapTienIch {
        string MaCN PK, FK
        string MaTI PK, FK
        date NgayBatDau PK
        date NgayKetThuc
    }
    NhanVien {
        string MaNV PK
        string HoTen
        string SDT
        string ChucVu
        decimal LuongCoBan
        string TrangThaiLamViec
    }
    LichSuCongTac {
        string MaLS PK
        string MaNV FK
        string MaCN FK
        string ChucVu
        date NgayBatDau
        date NgayKetThuc
        string LyDo
    }
    KyHoaHong {
        string MaCN PK, FK
        date Thang PK
        decimal MucHoaHongChot
        datetime ThoiDiemChot
        string MaNVChot FK
    }
    ChiTietHoaHong {
        string MaLichHen PK, FK
        string MaCN FK
        date Thang FK
    }
    LichHenPT {
        string MaLichHen PK
        string MaNV
        string MaCN
        datetime NgayGio
        string TrangThai
    }
```

FK kỳ của ChiTietHoaHong là cặp (MaCN, Thang); AK lịch sử là (MaNV, NgayBatDau). LichHenPT thuộc C, chỉ hiện phần tham chiếu. Ràng buộc chức vụ theo ngày, khoảng thời gian và bảo vệ kỳ chốt nằm trong thiết kế logic.
