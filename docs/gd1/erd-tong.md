# ERD tổng GĐ1 — A/B/C/D

Bản ghép hiện hành ngày 08/11/2026: 20 thực thể/quan hệ, mỗi thực thể chỉ xuất hiện một lần và lấy thuộc tính từ module sở hữu. Kiểu trên sơ đồ là miền logic, không phải khai báo SQL. [Lược đồ, khóa và ràng buộc tổng](thiet-ke-tong.md).

```mermaid
erDiagram
    ChiNhanh ||--o{ CungCapTienIch : cung_cap
    TienIch ||--o{ CungCapTienIch : duoc_cung_cap
    NhanVien ||--|{ LichSuCongTac : lich_su
    ChiNhanh ||--o{ LichSuCongTac : noi_cong_tac
    ChiNhanh ||--o{ KyHoaHong : ky_chot
    NhanVien ||..o{ KyHoaHong : nguoi_chot
    KyHoaHong ||--o{ ChiTietHoaHong : gom
    LichHenPT ||..o| ChiTietHoaHong : tinh_hoa_hong
    ChiNhanh ||..o{ HoiVien : noi_dang_ky
    HoiVien ||..o{ HoaDon : mua
    NhanVien ||..o{ HoaDon : lap
    ChiNhanh ||..o{ HoaDon : noi_ban
    HoaDon ||--|{ ChiTietHoaDon : gom
    GoiDichVu |o..o{ ChiTietHoaDon : dong_goi
    SanPham |o..o{ ChiTietHoaDon : dong_san_pham
    ChiTietHoaDon ||..o{ GoiHoiVien : don_vi_goi
    HoaDon ||--|| GhiNhanLoyalty : ghi_nhan
    GoiHoiVien ||..o{ LichHenPT : cap_buoi
    NhanVien ||..o{ LichHenPT : day
    ChiNhanh ||..o{ LichHenPT : noi_tap
    LichHenPT ||..o| DanhGiaHLV : danh_gia
    ChiNhanh ||--|{ CuaKiemSoat : co_cua
    HoiVien ||..o{ CheckIn : vao_ra
    ChiNhanh ||..o{ CheckIn : noi_quet
    CuaKiemSoat ||..o{ CheckIn : cua_vao
    CuaKiemSoat |o..o{ CheckIn : cua_ra
    ChiNhanh ||--o{ TonKho : kho
    SanPham ||--o{ TonKho : ton
    TonKho ||..o{ GiaoDichKho : nhat_ky
    ChiNhanh ||..o{ GiaoDichKho : chi_nhanh
    SanPham ||..o{ GiaoDichKho : mat_hang
    HoaDon |o..o{ GiaoDichKho : xuat_ban
    NhanVien |o..o{ GiaoDichKho : ghi_nhan
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
    SanPham {
        string MaSP PK
        string TenSP
        string LoaiSP
        decimal GiaBan
        boolean DangKinhDoanh
    }
    TonKho {
        string MaCN PK, FK
        string MaSP PK, FK
        int SoLuongTon
    }
    HoaDon {
        string MaHD PK
        string MaHV FK
        string MaNV FK
        string MaCN FK
        datetime NgayLap
        decimal GiamGia
        decimal TongTien
        string HinhThucThanhToan
    }
    ChiTietHoaDon {
        string MaHD PK, FK
        int STT PK
        string LoaiDong
        string MaGoi FK
        string MaSP FK
        int SoLuong
        decimal DonGia
    }
    GiaoDichKho {
        string MaGDKho PK
        string MaCN FK
        string MaSP FK
        string MaNV FK
        string MaHD FK
        string LoaiGiaoDich
        int SoLuongThayDoi
        datetime ThoiDiem
    }
```

Bản xuất đã kiểm tra cú pháp và hiển thị bằng Mermaid 10.9.3: [ERD tổng dạng SVG để phóng to](erd-tong.svg).

## Cách đọc và ràng buộc ngoài hình

- Mỗi hóa đơn trong dữ liệu đã thanh toán có ít nhất một dòng và đúng một ghi nhận Loyalty; bản nháp chưa thanh toán không phải bản ghi hoàn tất. Sơ đồ B trước khi ghép dùng 0..1 cho quá trình ghi nhận; bản tổng dùng 1..1 ở trạng thái đã hoàn tất.
- Dòng hóa đơn tham chiếu đúng một trong GoiDichVu/SanPham. Dòng gói tạo đúng SoLuong đơn vị GoiHoiVien; dòng sản phẩm tạo 0. Hai cạnh tùy chọn không cho phép cả hai cùng có hoặc cùng rỗng.
- FK ghép: GoiHoiVien.(MaHD,STT) → ChiTietHoaDon; ChiTietHoaHong.(MaCN,Thang) → KyHoaHong; hai cửa vào/ra của CheckIn dùng (MaCN,SoHieuCuaVao/Ra); GiaoDichKho.(MaCN,MaSP) → TonKho.
- Cửa ra có thể rỗng; có TGRa và không có cửa ra biểu thị tự đóng theo quy ước mô phỏng. GiaoDichKho.MaHD/MaNV có thể rỗng khi không gắn hóa đơn/người ghi; bán hàng phải gắn hóa đơn.
- AK ngoài ký hiệu PK: HoiVien.SDT, HoiVien.CCCD; LichSuCongTac.(MaNV,NgayBatDau); GoiHoiVien.(MaHD,STT,DonVi); DanhGiaHLV.MaLichHen.
- Phân cấp nhân viên loại trừ/toàn phần ánh xạ vào ChucVu; lịch sử lưu ChucVu của từng giai đoạn. Không tạo bản sao thực thể NhanVien cho mỗi module.
- vGoiHoiVien là phép nối dẫn xuất, không phải thực thể lưu thêm. Không có FK CheckIn → lịch PT; PT không cấp quyền vào. Không có quầy kiểm tra dịch vụ.
- Điều kiện theo thời gian, chống trùng lịch, chủ gói, số buổi giữ, bất biến hóa đơn/hoa hồng và tính lấp đầy không thể hiện đủ bằng lực lượng quan hệ; xem bảng ràng buộc tổng.
