# Thiết kế mức logic — FitLife Hub (GĐ1)

Lược đồ CSDL quan hệ gồm **13 quan hệ**, hình vẽ tại [luoc-do-logic.png](luoc-do-logic.png) (nguồn [luoc-do-logic.drawio](luoc-do-logic.drawio)). Các quyết định nghiệp vụ làm căn cứ ở [quyet-dinh.md](../quyet-dinh.md); quy trình ở `a/`, `b/`, `c/`, `d/quy-trinh.md`.

Ký hiệu: <u>gạch chân</u> = khóa chính (PK); *nghiêng* = khóa ngoại (FK); AK = khóa ứng viên khác.

## 1. Lược đồ quan hệ và module sở hữu

| # | Quan hệ | Module |
|---|---|---|
| 1 | CHI_NHANH(<u>MaCN</u>, TenCN, DiaChi, SDT, GioMoCua, GioDongCua, SucChua) | A |
| 2 | NHAN_VIEN(<u>MaNV</u>, HoTen, SDT, ChucVu, *MaCNHienTai*, LuongCoBan) | A |
| 3 | LICH_SU_CONG_TAC(<u>MaLSCT</u>, *MaNV*, *MaCN*, NgayBatDau, NgayKetThuc) | A |
| 4 | HOA_HONG_CHI_NHANH(<u>MaHHCN</u>, *MaCN*, NgayBatDau, NgayKetThuc, MucHoaHongMoiBuoi) | A |
| 5 | HOI_VIEN(<u>MaHV</u>, HoTen, SDT, Email, CCCD, GioiTinh, NgaySinh, NgayDangKy, *MaCN*, TongChiTieu, DiemTichLuy, Hang) | B |
| 6 | HANG_MUC_CUNG_CAP(<u>MaHM</u>, TenHangMuc, LoaiHangMuc, Gia, ThoiHanThang, SoBuoiBanDau, DangKinhDoanh) | B (gói), D (sản phẩm) |
| 7 | GOI_HOI_VIEN(<u>MaGHV</u>, *MaHV*, *MaHM*, NgayKichHoat, NgayHetHan, SoBuoiConLai) | B |
| 8 | CHECK_IN(<u>MaCI</u>, *MaGHV*, *MaCN*, ThoiGianVao, ThoiGianRa, ThietBiVao, ThietBiRa) | C |
| 9 | LICH_HEN_PT(<u>MaLH</u>, *MaGHV*, *MaNV*, *MaCN*, NgayTap, ThoiGianBatDau, TrangThai) | C |
| 10 | DANH_GIA_HLV(<u>MaDG</u>, *MaLH*, Diem, NhanXet) | C |
| 11 | TON_KHO(<u>*MaCN*, *MaHM*</u>, SoLuongTon) | D |
| 12 | HOA_DON(<u>MaHD</u>, *MaHV*, *MaNV*, *MaCN*, NgayLap, GiamGia, TongTien, HinhThucThanhToan) | D |
| 13 | CHI_TIET_HOA_DON(<u>MaCTHD</u>, *MaHD*, *MaHM*, SoLuong, DonGia) | D |

## 2. Khóa, phụ thuộc hàm và dạng chuẩn từng quan hệ

Chỉ liệt kê phụ thuộc hàm (PTH) không hiển nhiên. Các giá trị "chụp lại lúc phát sinh" (đơn giá, ngày hết hạn) không phải PTH, vì danh mục có thể đổi theo thời gian.

| Quan hệ | Khóa | Tập PTH | Dạng chuẩn | Giải thích |
|---|---|---|---|---|
| CHI_NHANH | MaCN | MaCN → TenCN, DiaChi, SDT, GioMoCua, GioDongCua, SucChua | BCNF | Mọi vế trái là khóa |
| NHAN_VIEN | MaNV | MaNV → HoTen, SDT, ChucVu, MaCNHienTai, LuongCoBan | BCNF | `MaCNHienTai` được suy từ quan hệ **khác** (LICH_SU_CONG_TAC), không có PTH giữa các thuộc tính trong quan hệ này. Đây là dư thừa giữa các quan hệ, giữ khớp bằng R07. `ChucVu` là thuộc tính gốc, cố định |
| LICH_SU_CONG_TAC | MaLSCT; AK (MaNV, NgayBatDau) | MaLSCT → MaNV, MaCN, NgayBatDau, NgayKetThuc; (MaNV, NgayBatDau) → MaLSCT, MaCN, NgayKetThuc | BCNF | Các giai đoạn của một nhân viên không chồng nhau nên (MaNV, NgayBatDau) là khóa |
| HOA_HONG_CHI_NHANH | MaHHCN; AK (MaCN, NgayBatDau) | MaHHCN → MaCN, NgayBatDau, NgayKetThuc, MucHoaHongMoiBuoi; (MaCN, NgayBatDau) → phần còn lại | BCNF | Tương tự, các khoảng của một chi nhánh không chồng nhau |
| HOI_VIEN | MaHV; AK CCCD; AK SDT | MaHV → mọi thuộc tính; CCCD → mọi thuộc tính; SDT → mọi thuộc tính; **TongChiTieu → Hang** | **2NF** | Hạng được xác định từ tổng chi tiêu theo ngưỡng. `TongChiTieu` không phải khóa, nên MaHV → TongChiTieu → Hang là phụ thuộc bắc cầu, vi phạm 3NF. Đạt 2NF vì mọi khóa chỉ có một thuộc tính. **Cố ý giữ** `Hang` để giải vấn đề 4 |
| HANG_MUC_CUNG_CAP | MaHM | MaHM → TenHangMuc, LoaiHangMuc, Gia, ThoiHanThang, SoBuoiBanDau, DangKinhDoanh | BCNF | `LoaiHangMuc` không xác định giá hay thời hạn (nhiều gói cùng loại) |
| GOI_HOI_VIEN | MaGHV | MaGHV → MaHV, MaHM, NgayKichHoat, NgayHetHan, SoBuoiConLai | BCNF | `NgayHetHan` chụp lại lúc mua, không phải PTH của (MaHM, NgayKichHoat) vì thời hạn danh mục có thể đổi |
| CHECK_IN | MaCI | MaCI → MaGHV, MaCN, ThoiGianVao, ThoiGianRa, ThietBiVao, ThietBiRa | BCNF | Gói dùng ở mọi chi nhánh nên MaGHV không xác định MaCN. Tên thiết bị chỉ duy nhất **trong một chi nhánh** (ví dụ "Cổng 1"), nên ThietBiVao không xác định MaCN |
| LICH_HEN_PT | MaLH | MaLH → MaGHV, MaNV, MaCN, NgayTap, ThoiGianBatDau, TrangThai; **(MaNV, NgayTap) → MaCN** | **2NF** | PT chỉ dạy tại chi nhánh đang công tác và một ngày chỉ thuộc một chi nhánh (#12, #13), nên cùng PT cùng ngày thì cùng chi nhánh. (MaNV, NgayTap) không phải khóa vì một PT dạy nhiều buổi/ngày → vi phạm 3NF. **Cố ý giữ** `MaCN` để tính hoa hồng và báo cáo không phải nối lịch sử công tác. Bộ (MaNV, NgayTap, ThoiGianBatDau) không phải khóa vì lịch đã hủy có thể trùng |
| DANH_GIA_HLV | MaDG; AK MaLH | MaDG → MaLH, Diem, NhanXet; MaLH → MaDG, Diem, NhanXet | BCNF | Mỗi lịch tối đa một đánh giá |
| TON_KHO | (MaCN, MaHM) | (MaCN, MaHM) → SoLuongTon | BCNF | |
| HOA_DON | MaHD | MaHD → MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan; **(MaNV, NgayLap) → MaCN** | **2NF** | Chi nhánh của hóa đơn là chi nhánh nhân viên lập đang công tác tại ngày lập (#24), nên cùng nhân viên, cùng thời điểm lập thì cùng chi nhánh. (MaNV, NgayLap) không được khai báo là khóa → vi phạm 3NF. **Cố ý giữ** `MaCN` vì báo cáo doanh thu theo chi nhánh (vấn đề 3) là truy vấn chính. `TongTien` suy từ CHI_TIET_HOA_DON (quan hệ khác) nên không làm vi phạm dạng chuẩn, giữ khớp bằng R23 |
| CHI_TIET_HOA_DON | MaCTHD; AK (MaHD, MaHM) | MaCTHD → MaHD, MaHM, SoLuong, DonGia; (MaHD, MaHM) → MaCTHD, SoLuong, DonGia | BCNF | `DonGia` chụp lại lúc bán, không phụ thuộc MaHM. Thành tiền không lưu |

## 3. Kết luận dạng chuẩn toàn lược đồ

- 10 quan hệ đạt **BCNF**.
- HOI_VIEN, LICH_HEN_PT, HOA_DON đạt **2NF**, không đạt 3NF, do **cố ý lưu dư** `Hang`, `MaCN` (lịch hẹn), `MaCN` (hóa đơn) để các truy vấn tần suất cao hoặc nặng (thanh toán, hoa hồng, doanh thu theo chi nhánh) không phải tính lại hoặc nối thêm bảng.
- **Toàn lược đồ đạt 2NF** (dạng chuẩn của lược đồ là dạng chuẩn thấp nhất của các quan hệ).
- Muốn đạt 3NF thì tách `Hang` thành bảng ngưỡng hạng và bỏ hai cột `MaCN`; nhóm không chọn vì đi ngược mục tiêu tối ưu của vấn đề 3, 4, 5. Các dư thừa này được bảo vệ bằng ràng buộc R10, R18, R25.

## 4. Mô tả quan hệ và thuộc tính

Kiểu ở mức logic: *Mã* (chuỗi/số định danh), *Chuỗi*, *Số nguyên*, *Tiền* (số nguyên, đơn vị đồng), *Ngày*, *Giờ*, *Ngày giờ*, *Bit* (0/1). Ngày giờ theo Asia/Saigon. "Rỗng" = được phép NULL.

### CHI_NHANH — chi nhánh của chuỗi (12 dòng cố định)

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaCN | Mã | Không | Mã chi nhánh |
| TenCN, DiaChi, SDT | Chuỗi | Không | Tên, địa chỉ, số điện thoại |
| GioMoCua, GioDongCua | Giờ | Không | Giờ mở, đóng cửa trong cùng ngày; GioMoCua < GioDongCua |
| SucChua | Số nguyên | Không | Sức chứa > 0, chỉ dùng cho báo cáo lấp đầy |

### NHAN_VIEN — nhân viên

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaNV | Mã | Không | Mã nhân viên |
| HoTen, SDT | Chuỗi | Không | Họ tên, số điện thoại |
| ChucVu | Chuỗi | Không | ∈ {PT, Lễ tân, Quản lý chi nhánh, Nhân viên bán hàng}; gán lúc tiếp nhận, không đổi |
| MaCNHienTai | Mã | Có | Chi nhánh hiện tại, đồng bộ từ lịch sử mỗi ngày; rỗng = đã nghỉ việc |
| LuongCoBan | Tiền | Không | Lương hiện tại ≥ 0; không lưu lịch sử lương |

### LICH_SU_CONG_TAC — các giai đoạn công tác

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaLSCT | Mã | Không | Mã giai đoạn |
| MaNV, MaCN | Mã | Không | Nhân viên, chi nhánh trong giai đoạn |
| NgayBatDau | Ngày | Không | Ngày bắt đầu, tính cả ngày này |
| NgayKetThuc | Ngày | Có | Ngày kết thúc, tính cả ngày này; rỗng = chưa xác định; nếu có thì ≥ NgayBatDau |

### HOA_HONG_CHI_NHANH — mức hoa hồng mỗi buổi PT theo khoảng ngày

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaHHCN | Mã | Không | Mã khoảng mức |
| MaCN | Mã | Không | Chi nhánh |
| NgayBatDau, NgayKetThuc | Ngày | Không / Có | Khoảng hiệu lực, tính cả hai biên; kết thúc rỗng = đang áp dụng |
| MucHoaHongMoiBuoi | Tiền | Không | Số tiền hoa hồng cho một buổi Đã hoàn thành, ≥ 0 |

### HOI_VIEN — hội viên

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaHV | Mã | Không | Mã hội viên |
| HoTen | Chuỗi | Không | Họ tên |
| SDT | Chuỗi | Không | Số điện thoại, duy nhất |
| Email | Chuỗi | Có | Email |
| CCCD | Chuỗi | Không | 12 chữ số, duy nhất |
| GioiTinh | Chuỗi | Không | ∈ {Nam, Nữ, Khác} |
| NgaySinh | Ngày | Không | Trước ngày đăng ký |
| NgayDangKy | Ngày | Không | Ngày tạo hồ sơ |
| MaCN | Mã | Không | Chi nhánh đăng ký |
| TongChiTieu | Tiền | Không | Tổng `TongTien` các hóa đơn trong năm dương lịch hiện tại, ≥ 0 |
| DiemTichLuy | Số nguyên | Không | Tổng điểm mọi hóa đơn, ≥ 0, không reset |
| Hang | Chuỗi | Không | ∈ {Silver, Gold, Platinum}, theo ngưỡng 20/50 triệu của TongChiTieu |

### HANG_MUC_CUNG_CAP — danh mục gói dịch vụ và sản phẩm

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaHM | Mã | Không | Mã hàng mục |
| TenHangMuc | Chuỗi | Không | Tên; với sản phẩm ghi rõ nhóm (Whey, bình nước, thảm yoga…) |
| LoaiHangMuc | Chuỗi | Không | ∈ {Gym, Yoga, PT, Spa, Vé lượt, Sản phẩm} |
| Gia | Tiền | Không | Giá bán hiện tại > 0 |
| ThoiHanThang | Số nguyên | Có | Gym/Yoga ∈ {1, 3, 6, 12}; PT ∈ {3, 6, 12}; loại khác rỗng |
| SoBuoiBanDau | Số nguyên | Có | PT > 0; Spa, Vé lượt = 1; loại khác rỗng |
| DangKinhDoanh | Bit | Không | 1 = đang bán, 0 = ngừng |

### GOI_HOI_VIEN — gói hội viên đã mua (mỗi dòng một mã QR)

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaGHV | Mã | Không | Mã gói, đồng thời là mã QR quét cổng |
| MaHV | Mã | Không | Hội viên sở hữu |
| MaHM | Mã | Không | Hàng mục loại gói (không phải Sản phẩm) |
| NgayKichHoat | Ngày | Không | Ngày mua |
| NgayHetHan | Ngày | Không | Gym/Yoga/PT: ngày mua + số tháng − 1 ngày; Spa, Vé lượt: ngày mua + 6 ngày |
| SoBuoiConLai | Số nguyên | Có | Gym/Yoga rỗng; PT, Spa, Vé lượt: 0 … SoBuoiBanDau |

### CHECK_IN — một phiên vào/ra

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaCI | Mã | Không | Mã phiên |
| MaGHV | Mã | Không | Gói đã quét để vào |
| MaCN | Mã | Không | Chi nhánh |
| ThoiGianVao | Ngày giờ | Không | Lúc quét vào |
| ThoiGianRa | Ngày giờ | Có | Lúc ra; rỗng = đang trong phòng; nếu có thì > ThoiGianVao |
| ThietBiVao | Chuỗi | Không | Tên thiết bị quét vào, duy nhất trong chi nhánh |
| ThietBiRa | Chuỗi | Có | Tên thiết bị quét ra; rỗng khi đã có ThoiGianRa = phiên bị tự đóng, giờ ra ước tính |

### LICH_HEN_PT — lịch hẹn PT một giờ

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaLH | Mã | Không | Mã lịch |
| MaGHV | Mã | Không | Gói PT dùng cho buổi này (suy ra hội viên) |
| MaNV | Mã | Không | PT dạy |
| MaCN | Mã | Không | Chi nhánh dạy |
| NgayTap | Ngày | Không | Ngày tập |
| ThoiGianBatDau | Giờ | Không | Giờ tròn; buổi kéo dài 1 giờ, nằm trong giờ mở cửa |
| TrangThai | Số nguyên | Không | 0 = Đã đặt (mặc định), 1 = Hủy, 2 = Đã hoàn thành |

### DANH_GIA_HLV — đánh giá của hội viên cho một buổi PT

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaDG | Mã | Không | Mã đánh giá |
| MaLH | Mã | Không | Lịch được đánh giá, duy nhất |
| Diem | Số nguyên | Không | 1 … 5 |
| NhanXet | Chuỗi | Có | Nhận xét |

### TON_KHO — tồn kho sản phẩm tại chi nhánh

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaCN, MaHM | Mã | Không | Chi nhánh, sản phẩm; có dòng = chi nhánh đang bán sản phẩm này |
| SoLuongTon | Số nguyên | Không | ≥ 0 |

### HOA_DON — hóa đơn

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaHD | Mã | Không | Mã hóa đơn |
| MaHV | Mã | Không | Hội viên mua |
| MaNV | Mã | Không | Nhân viên lập |
| MaCN | Mã | Không | Chi nhánh lập = chi nhánh của nhân viên tại ngày lập |
| NgayLap | Ngày giờ | Không | Thời điểm thanh toán |
| GiamGia | Tiền | Không | 0 ≤ GiamGia ≤ tạm tính |
| TongTien | Tiền | Không | Tạm tính − GiamGia, ≥ 0; lưu sẵn |
| HinhThucThanhToan | Chuỗi | Không | ∈ {Tiền mặt, VNPay, Thẻ} |

### CHI_TIET_HOA_DON — dòng hóa đơn

| Thuộc tính | Kiểu | Rỗng | Ý nghĩa / miền |
|---|---|---|---|
| MaCTHD | Mã | Không | Mã dòng |
| MaHD | Mã | Không | Hóa đơn |
| MaHM | Mã | Không | Hàng mục (gói hoặc sản phẩm); (MaHD, MaHM) duy nhất |
| SoLuong | Số nguyên | Không | > 0 |
| DonGia | Tiền | Không | Giá của hàng mục tại lúc bán, > 0 |

## 5. Khóa ngoại (20)

| # | Khóa ngoại | Tham chiếu | Bản số |
|---|---|---|---|
| 1 | NHAN_VIEN.MaCNHienTai | CHI_NHANH.MaCN | n–0..1 (rỗng khi nghỉ việc) |
| 2 | LICH_SU_CONG_TAC.MaNV | NHAN_VIEN.MaNV | n–1, mỗi nhân viên ≥ 1 giai đoạn |
| 3 | LICH_SU_CONG_TAC.MaCN | CHI_NHANH.MaCN | n–1 |
| 4 | HOA_HONG_CHI_NHANH.MaCN | CHI_NHANH.MaCN | n–1 |
| 5 | HOI_VIEN.MaCN | CHI_NHANH.MaCN | n–1 |
| 6 | GOI_HOI_VIEN.MaHV | HOI_VIEN.MaHV | n–1 |
| 7 | GOI_HOI_VIEN.MaHM | HANG_MUC_CUNG_CAP.MaHM | n–1 |
| 8 | CHECK_IN.MaGHV | GOI_HOI_VIEN.MaGHV | n–1 |
| 9 | CHECK_IN.MaCN | CHI_NHANH.MaCN | n–1 |
| 10 | LICH_HEN_PT.MaGHV | GOI_HOI_VIEN.MaGHV | n–1 |
| 11 | LICH_HEN_PT.MaNV | NHAN_VIEN.MaNV | n–1 |
| 12 | LICH_HEN_PT.MaCN | CHI_NHANH.MaCN | n–1 |
| 13 | DANH_GIA_HLV.MaLH | LICH_HEN_PT.MaLH | 0..1–1 |
| 14 | TON_KHO.MaCN | CHI_NHANH.MaCN | n–1 |
| 15 | TON_KHO.MaHM | HANG_MUC_CUNG_CAP.MaHM | n–1 |
| 16 | HOA_DON.MaHV | HOI_VIEN.MaHV | n–1 |
| 17 | HOA_DON.MaNV | NHAN_VIEN.MaNV | n–1 |
| 18 | HOA_DON.MaCN | CHI_NHANH.MaCN | n–1 |
| 19 | CHI_TIET_HOA_DON.MaHD | HOA_DON.MaHD | n–1, mỗi hóa đơn ≥ 1 dòng |
| 20 | CHI_TIET_HOA_DON.MaHM | HANG_MUC_CUNG_CAP.MaHM | n–1 |

Không xóa dây chuyền (cascade) dữ liệu lịch sử. Bảng cha đã được tham chiếu thì không xóa.

## 6. Ràng buộc toàn vẹn

Mỗi ràng buộc gồm **bối cảnh** (các quan hệ liên quan), **nội dung**, và **bảng tầm ảnh hưởng**: `+` là thao tác có thể vi phạm nên phải kiểm tra, `−` là không vi phạm; ghi trong ngoặc các thuộc tính mà thao tác Sửa cần kiểm tra. Ràng buộc khóa chính, khóa ứng viên và 20 khóa ngoại ở mục 2, 5 không lặp lại ở đây.

### 6.1. Ràng buộc miền giá trị

**R01** — Bối cảnh: CHI_NHANH. Nội dung: ∀c ∈ CHI_NHANH: c.GioMoCua < c.GioDongCua ∧ c.SucChua > 0.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| CHI_NHANH | + | − | + (GioMoCua, GioDongCua, SucChua) |

**R02** — Bối cảnh: NHAN_VIEN, LICH_SU_CONG_TAC. Nội dung: NHAN_VIEN.ChucVu ∈ {PT, Lễ tân, Quản lý chi nhánh, Nhân viên bán hàng} và không đổi sau khi tạo; NHAN_VIEN.LuongCoBan ≥ 0; LICH_SU_CONG_TAC.NgayKetThuc rỗng hoặc ≥ NgayBatDau.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| NHAN_VIEN | + | − | + (ChucVu, LuongCoBan) |
| LICH_SU_CONG_TAC | + | − | + (NgayBatDau, NgayKetThuc) |

**R03** — Bối cảnh: HOA_HONG_CHI_NHANH. Nội dung: MucHoaHongMoiBuoi ≥ 0; NgayKetThuc rỗng hoặc ≥ NgayBatDau.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOA_HONG_CHI_NHANH | + | − | + (MucHoaHongMoiBuoi, NgayBatDau, NgayKetThuc) |

**R04** — Bối cảnh: HOI_VIEN. Nội dung: CCCD gồm 12 chữ số; GioiTinh ∈ {Nam, Nữ, Khác}; NgaySinh < NgayDangKy; TongChiTieu ≥ 0; DiemTichLuy ≥ 0; Hang ∈ {Silver, Gold, Platinum}.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOI_VIEN | + | − | + (CCCD, GioiTinh, NgaySinh, NgayDangKy, TongChiTieu, DiemTichLuy, Hang) |

**R05** — Bối cảnh: HANG_MUC_CUNG_CAP. Nội dung: Gia > 0; LoaiHangMuc ∈ {Gym, Yoga, PT, Spa, Vé lượt, Sản phẩm}; Gym/Yoga: ThoiHanThang ∈ {1, 3, 6, 12}, SoBuoiBanDau rỗng; PT: ThoiHanThang ∈ {3, 6, 12}, SoBuoiBanDau > 0; Spa, Vé lượt: ThoiHanThang rỗng, SoBuoiBanDau = 1; Sản phẩm: cả hai rỗng.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HANG_MUC_CUNG_CAP | + | − | + (Gia, LoaiHangMuc, ThoiHanThang, SoBuoiBanDau) |

**R06** — Bối cảnh: CHECK_IN, CHI_TIET_HOA_DON, HOA_DON, TON_KHO, DANH_GIA_HLV, LICH_HEN_PT. Nội dung: CHECK_IN.ThoiGianRa rỗng hoặc > ThoiGianVao; CHI_TIET_HOA_DON.SoLuong > 0, DonGia > 0; HOA_DON.HinhThucThanhToan ∈ {Tiền mặt, VNPay, Thẻ}; TON_KHO.SoLuongTon ≥ 0; DANH_GIA_HLV.Diem ∈ 1..5; LICH_HEN_PT.TrangThai ∈ {0, 1, 2}.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| CHECK_IN | + | − | + (ThoiGianVao, ThoiGianRa) |
| CHI_TIET_HOA_DON | + | − | + (SoLuong, DonGia) |
| HOA_DON | + | − | + (HinhThucThanhToan) |
| TON_KHO | + | − | + (SoLuongTon) |
| DANH_GIA_HLV | + | − | + (Diem) |
| LICH_HEN_PT | + | − | + (TrangThai) |

### 6.2. Ràng buộc liên thuộc tính, liên bộ và liên quan hệ — Module A

**R07** — Bối cảnh: NHAN_VIEN, LICH_SU_CONG_TAC. Nội dung: sau khi đồng bộ đầu ngày, với mỗi nhân viên: nếu có giai đoạn g bao phủ hôm nay thì NHAN_VIEN.MaCNHienTai = g.MaCN; nếu không có thì MaCNHienTai rỗng.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| NHAN_VIEN | + | − | + (MaCNHienTai) |
| LICH_SU_CONG_TAC | + | + | + (MaNV, MaCN, NgayBatDau, NgayKetThuc) |

**R08** — Bối cảnh: LICH_SU_CONG_TAC. Nội dung: hai giai đoạn khác nhau của cùng một nhân viên không có ngày chung (khoảng `[NgayBatDau, NgayKetThuc]` không giao nhau, NgayKetThuc rỗng coi là vô hạn).

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_SU_CONG_TAC | + | − | + (MaNV, NgayBatDau, NgayKetThuc) |

**R09** — Bối cảnh: HOA_HONG_CHI_NHANH. Nội dung: hai khoảng mức khác nhau của cùng một chi nhánh không có ngày chung.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOA_HONG_CHI_NHANH | + | − | + (MaCN, NgayBatDau, NgayKetThuc) |

### 6.3. Module B

**R10** — Bối cảnh: HOI_VIEN. Nội dung: Hang = Silver nếu TongChiTieu < 20.000.000; Gold nếu 20.000.000 ≤ TongChiTieu < 50.000.000; Platinum nếu TongChiTieu ≥ 50.000.000.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOI_VIEN | + | − | + (TongChiTieu, Hang) |

**R11** — Bối cảnh: HOI_VIEN, HOA_DON. Nội dung: TongChiTieu của hội viên = tổng TongTien các hóa đơn của hội viên có NgayLap thuộc năm hiện tại; DiemTichLuy = tổng ⌊TongTien / 100.000⌋ trên mọi hóa đơn của hội viên.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOI_VIEN | + | − | + (TongChiTieu, DiemTichLuy) |
| HOA_DON | + | + | + (MaHV, TongTien, NgayLap) |

(Xóa/sửa hóa đơn bị cấm bởi R24; job đầu năm tính lại TongChiTieu.)

**R12** — Bối cảnh: GOI_HOI_VIEN, HANG_MUC_CUNG_CAP. Nội dung: hàng mục của gói không phải Sản phẩm. NgayHetHan = NgayKichHoat + ThoiHanThang tháng − 1 ngày (Gym, Yoga, PT; tháng đích thiếu ngày thì lấy cuối tháng) hoặc NgayKichHoat + 6 ngày (Spa, Vé lượt), theo thời hạn tại lúc mua. SoBuoiConLai rỗng với Gym/Yoga; 0 ≤ SoBuoiConLai ≤ SoBuoiBanDau với loại còn lại.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| GOI_HOI_VIEN | + | − | + (MaHM, NgayKichHoat, NgayHetHan, SoBuoiConLai) |
| HANG_MUC_CUNG_CAP | − | − | + (LoaiHangMuc) |

**R13** — Bối cảnh: GOI_HOI_VIEN, HOA_DON, CHI_TIET_HOA_DON. Nội dung: mỗi dòng hóa đơn loại gói có SoLuong = n sinh đúng n gói cùng MaHV của hóa đơn, cùng MaHM, NgayKichHoat = ngày của NgayLap; gói chỉ được tạo trong giao dịch thanh toán.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| GOI_HOI_VIEN | + | + | + (MaHV, MaHM, NgayKichHoat) |
| CHI_TIET_HOA_DON | + | + | + (MaHM, SoLuong) |
| HOA_DON | − | + | + (MaHV, NgayLap) |

### 6.4. Module C

**R14** — Bối cảnh: CHECK_IN, GOI_HOI_VIEN, HANG_MUC_CUNG_CAP. Nội dung: tại thời điểm thêm phiên, gói có NgayKichHoat ≤ ngày(ThoiGianVao) ≤ NgayHetHan, SoBuoiConLai rỗng hoặc > 0, và loại hàng mục ≠ PT. Nếu gói giới hạn lượt thì SoBuoiConLai giảm 1 cùng giao dịch.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| CHECK_IN | + | − | + (MaGHV, ThoiGianVao) |
| GOI_HOI_VIEN | − | − | + (SoBuoiConLai) |

**R15** — Bối cảnh: LICH_HEN_PT, CHI_NHANH. Nội dung: ThoiGianBatDau là giờ tròn (phút = giây = 0); GioMoCua ≤ ThoiGianBatDau và ThoiGianBatDau + 1 giờ ≤ GioDongCua của chi nhánh lịch.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (MaCN, ThoiGianBatDau) |
| CHI_NHANH | − | − | + (GioMoCua, GioDongCua) |

**R16** — Bối cảnh: LICH_HEN_PT. Nội dung (chống trùng lịch PT — vấn đề 2): không có hai lịch có TrangThai ≠ 1 cùng (MaNV, NgayTap, ThoiGianBatDau).

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (MaNV, NgayTap, ThoiGianBatDau, TrangThai) |

**R17** — Bối cảnh: LICH_HEN_PT, GOI_HOI_VIEN. Nội dung: không có hai lịch có TrangThai ≠ 1 của cùng một hội viên (MaHV của gói) cùng NgayTap, ThoiGianBatDau.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (MaGHV, NgayTap, ThoiGianBatDau, TrangThai) |
| GOI_HOI_VIEN | − | − | + (MaHV) |

**R18** — Bối cảnh: LICH_HEN_PT, NHAN_VIEN, LICH_SU_CONG_TAC. Nội dung: với mỗi lịch, NHAN_VIEN.ChucVu của MaNV = PT, và tồn tại đúng một giai đoạn của MaNV bao phủ NgayTap có MaCN = LICH_HEN_PT.MaCN.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (MaNV, MaCN, NgayTap) |
| NHAN_VIEN | − | − | + (ChucVu) |
| LICH_SU_CONG_TAC | + | + | + (MaNV, MaCN, NgayBatDau, NgayKetThuc) |

**R19** — Bối cảnh: LICH_HEN_PT, GOI_HOI_VIEN, HANG_MUC_CUNG_CAP. Nội dung: gói của lịch có loại PT và NgayKichHoat ≤ NgayTap ≤ NgayHetHan; với mỗi gói PT, số lịch có TrangThai = 0 ≤ SoBuoiConLai.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (MaGHV, NgayTap, TrangThai) |
| GOI_HOI_VIEN | − | − | + (NgayHetHan, SoBuoiConLai) |

**R20** — Bối cảnh: LICH_HEN_PT, GOI_HOI_VIEN. Nội dung: lịch mới có TrangThai = 0; chỉ được chuyển 0 → 1 khi thời điểm hủy ≤ (NgayTap + ThoiGianBatDau) − 2 giờ, và 0 → 2 khi đã tới giờ tập, đồng thời SoBuoiConLai của gói giảm đúng 1. Không có chuyển trạng thái nào khác.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | + | − | + (TrangThai) |
| GOI_HOI_VIEN | − | − | + (SoBuoiConLai) |

**R21** — Bối cảnh: LICH_HEN_PT. Nội dung: lịch có NgayTap thuộc tháng đã qua không được sửa hoặc xóa (bảo đảm hoa hồng tháng cũ không đổi).

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| LICH_HEN_PT | − | + | + (MaGHV, MaNV, MaCN, NgayTap, ThoiGianBatDau, TrangThai) |

**R22** — Bối cảnh: DANH_GIA_HLV, LICH_HEN_PT. Nội dung: lịch được đánh giá có TrangThai = 2.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| DANH_GIA_HLV | + | − | + (MaLH) |
| LICH_HEN_PT | − | − | + (TrangThai) |

### 6.5. Module D

**R23** — Bối cảnh: HOA_DON, CHI_TIET_HOA_DON. Nội dung: mỗi hóa đơn có ít nhất một dòng; gọi TamTinh = Σ SoLuong × DonGia các dòng của hóa đơn, thì 0 ≤ GiamGia ≤ TamTinh và TongTien = TamTinh − GiamGia.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOA_DON | + | − | + (GiamGia, TongTien) |
| CHI_TIET_HOA_DON | + | + | + (MaHD, SoLuong, DonGia) |

**R24** — Bối cảnh: HOA_DON, CHI_TIET_HOA_DON. Nội dung: hóa đơn đã thanh toán và các dòng của nó không được thêm dòng, sửa hoặc xóa.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOA_DON | − | + | + (mọi thuộc tính) |
| CHI_TIET_HOA_DON | + | + | + (mọi thuộc tính) |

**R25** — Bối cảnh: HOA_DON, LICH_SU_CONG_TAC. Nội dung: HOA_DON.MaCN = chi nhánh của giai đoạn công tác của MaNV bao phủ ngày(NgayLap).

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| HOA_DON | + | − | + (MaNV, MaCN, NgayLap) |
| LICH_SU_CONG_TAC | − | + | + (MaNV, MaCN, NgayBatDau, NgayKetThuc) |

**R26** — Bối cảnh: TON_KHO, HANG_MUC_CUNG_CAP. Nội dung: hàng mục trong tồn kho có loại Sản phẩm.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| TON_KHO | + | − | + (MaHM) |
| HANG_MUC_CUNG_CAP | − | − | + (LoaiHangMuc) |

**R27** — Bối cảnh: TON_KHO, CHI_TIET_HOA_DON, HOA_DON. Nội dung: mỗi dòng sản phẩm của hóa đơn có dòng TON_KHO(MaCN của hóa đơn, MaHM) tại lúc bán, và SoLuongTon giảm đúng SoLuong trong cùng giao dịch, không âm.

| | Thêm | Xóa | Sửa |
|---|---|---|---|
| CHI_TIET_HOA_DON | + | − | + (MaHM, SoLuong) |
| TON_KHO | − | + | + (SoLuongTon) |

## 7. Ràng buộc kiểm tra tại thời điểm nghiệp vụ

Một số điều kiện chỉ đúng **tại lúc thao tác** chứ không phải mọi lúc, nên không viết thành ràng buộc trạng thái ở trên mà được kiểm tra trong thủ tục ở GĐ2:
- Gói còn hiệu lực khi quét vào (R14) — gói hết hạn về sau không làm phiên cũ sai.
- Hàng mục đang kinh doanh khi lập hóa đơn — ngừng bán về sau không làm hóa đơn cũ sai.
- Điều chuyển/nghỉ việc chỉ được chấp nhận khi không làm lịch Đã đặt từ ngày hiệu lực vi phạm R18 (QT-A03).
- Mức hoa hồng mới chỉ bắt đầu từ ngày mai (QT-A07).
- Ba thao tác ghi gộp trong một giao dịch: thanh toán (R11, R13, R23, R27), quét vào (R14), job tới giờ (R20).
