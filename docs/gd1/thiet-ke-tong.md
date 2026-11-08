# Thiết kế tổng GĐ1 — A/B/C/D

Phạm vi: quy trình, tần suất, ERD, lược đồ quan hệ, khóa/phụ thuộc hàm/dạng chuẩn, thuộc tính và ràng buộc. Không có script SQL, index, partition, thủ tục, ma trận truy vấn hoặc đo hiệu năng. [ERD tổng](erd-tong.md), [quyết định chung](../quyet-dinh.md), [sổ 17 mục rà soát](theo-doi-ra-soat.md).

## Lược đồ và quyền sở hữu

Mỗi quan hệ có một module sở hữu. Bảng tham chiếu xuất hiện trong ERD module khác chỉ là phần nhìn, không phải bảng mới. Các tên cột cùng nghĩa và khóa tham chiếu dùng cùng miền logic; độ dài/kiểu SQL chốt ở GĐ2.

### Module A

```text
ChiNhanh(MaCN, TenCN, DiaChi, SDT, GioMoCua, GioDongCua, SucChua, HoaHongMoiBuoi, TrangThai)
TienIch(MaTI, TenTI, Loai, MoTa, TrangThai)
CungCapTienIch(MaCN, MaTI, NgayBatDau, NgayKetThuc)
NhanVien(MaNV, HoTen, SDT, ChucVu, LuongCoBan, TrangThaiLamViec)
LichSuCongTac(MaLS, MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc, LyDo)
KyHoaHong(MaCN, Thang, MucHoaHongChot, ThoiDiemChot, MaNVChot)
ChiTietHoaHong(MaLichHen, MaCN, Thang)
```

### Module B

```text
HoiVien(MaHV, HoTen, SDT, Email, CCCD, GioiTinh, NgaySinh, NgayDangKy, MaCNDangKy, TongChiTieu12T, DiemTichLuy, Hang)
GoiDichVu(MaGoi, TenGoi, LoaiGoi, CachTinh, ThoiHanThang, SoBuoiBanDau, Gia, DangKinhDoanh)
GoiHoiVien(MaGHV, MaHD, STT, DonVi, NgayBatDau, NgayHetHan, SoBuoiConLai)
GhiNhanLoyalty(MaHD, DiemCong, ThoiDiemGhiNhan, DaLoaiKhoi12T)
```

### Module C

```text
CuaKiemSoat(MaCN, SoHieuCua, ChucNang, DangHoatDong)
CheckIn(MaCheckIn, MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa)
LichHenPT(MaLichHen, MaGHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc)
DanhGiaHLV(MaDanhGia, MaLichHen, Diem, NhanXet, ThoiDiemDanhGia)
```

### Module D

```text
SanPham(MaSP, TenSP, LoaiSP, GiaBan, DangKinhDoanh)
TonKho(MaCN, MaSP, SoLuongTon)
HoaDon(MaHD, MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan)
ChiTietHoaDon(MaHD, STT, LoaiDong, MaGoi, MaSP, SoLuong, DonGia)
GiaoDichKho(MaGDKho, MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem)
```

## Khóa, phụ thuộc hàm và dạng chuẩn từng quan hệ

PK là khóa chính, AK là khóa ứng viên khác. “Mỗi khóa → phần còn lại” là tập PTH tối thiểu theo các khóa công bố, không mặc định tên/ngày/số điện thoại nhân viên duy nhất.

| Quan hệ | PK / AK | PTH và dạng chuẩn |
|---|---|---|
| ChiNhanh | MaCN; không có khóa khác được giả định | MaCN → phần còn lại; BCNF |
| TienIch | MaTI | MaTI → phần còn lại; BCNF |
| CungCapTienIch | (MaCN, MaTI, NgayBatDau) | Khóa → NgayKetThuc; BCNF |
| NhanVien | MaNV | MaNV → phần còn lại; BCNF theo PTH nội quan hệ này |
| LichSuCongTac | MaLS; AK (MaNV, NgayBatDau) | Mỗi khóa → phần còn lại; BCNF |
| KyHoaHong | (MaCN, Thang) | Khóa → phần còn lại; BCNF |
| ChiTietHoaHong | MaLichHen | MaLichHen → MaCN, Thang; BCNF |
| HoiVien | PK MaHV; AK CCCD, SDT (bắt buộc, duy nhất theo QT-B1) | Mỗi khóa → phần còn lại; TongChiTieu12T → Hang theo ngưỡng cố định. Đạt 2NF, không đạt 3NF/BCNF; lưu hạng dư thừa có chủ đích để đọc nhanh theo GD1-09 |
| GoiDichVu | PK MaGoi | MaGoi → phần còn lại; BCNF theo PTH đã nêu; kiểm tra tổ hợp loại/cách tính bằng ràng buộc miền |
| GoiHoiVien | PK MaGHV; AK (MaHD, STT, DonVi) | Mỗi khóa → phần còn lại; BCNF theo PTH đã nêu |
| GhiNhanLoyalty | PK MaHD | MaHD → phần còn lại; BCNF; DiemCong tính từ tiền ở bảng khác, không tạo PTH ngoài khóa trong quan hệ này |
| CuaKiemSoat | PK `{MaCN, SoHieuCua}` | `(MaCN, SoHieuCua) → ChucNang, DangHoatDong`; BCNF. Số hiệu cửa chỉ duy nhất trong phạm vi một chi nhánh; mọi định thức không tầm thường là khóa. |
| CheckIn | PK `{MaCheckIn}` | `MaCheckIn → MaHV, MaCN, SoHieuCuaVao, SoHieuCuaRa, TGVao, TGRa`; BCNF. “Một lượt đang mở/hội viên” là ràng buộc có điều kiện, không phải PTH trên toàn quan hệ. Không lưu mô tả hội viên/chi nhánh/cửa. |
| LichHenPT | PK `{MaLichHen}` | `MaLichHen → MaGHV, MaNV, MaCN, NgayGio, ThoiDiemDat, TrangThai, DaTruBuoi, LyDoKetThuc`; BCNF theo PTH đã nêu. Không lưu MaHV vì MaGHV xác định chủ gói; nhờ đó không tạo PTH không khóa MaGHV → MaHV trong lịch. Tính duy nhất `(MaNV, NgayGio)` chỉ áp dụng cho dòng chưa hủy nên là ràng buộc có điều kiện, không coi là khóa ứng viên toàn quan hệ. |
| DanhGiaHLV | PK `{MaDanhGia}`; AK `{MaLichHen}` | `MaDanhGia → MaLichHen, Diem, NhanXet, ThoiDiemDanhGia`; `MaLichHen → MaDanhGia, Diem, NhanXet, ThoiDiemDanhGia`; BCNF. `MaLichHen` là khóa ứng viên `UNIQUE`, bảo đảm một lịch có tối đa một đánh giá. Không lặp `MaHV`, `MaNV` vì có thể suy ra qua lịch. |
| SanPham | PK `{MaSP}` | `MaSP → TenSP, LoaiSP, GiaBan, DangKinhDoanh`; BCNF: vế trái của mọi PTH không tầm thường là khóa. Tên sản phẩm không mặc định duy nhất. |
| TonKho | PK `{MaCN, MaSP}` | `(MaCN, MaSP) → SoLuongTon`; BCNF. Số dư tách riêng cho từng chi nhánh và mặt hàng. |
| HoaDon | PK `{MaHD}` | `MaHD → MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan`; BCNF. `TongTien` là tổng chụp tại thời điểm bán, được kiểm tra với chi tiết; không suy ra bằng PTH đơn trong cùng quan hệ. |
| ChiTietHoaDon | PK `{MaHD, STT}` | `(MaHD, STT) → LoaiDong, MaGoi, MaSP, SoLuong, DonGia`; BCNF theo PTH đã nêu. Không lưu tên sản phẩm/gói hoặc ThanhTien dẫn xuất. (SoLuong, DonGia) → ThanhTien chỉ thuộc kết quả truy vấn mở rộng, không phải PTH của bảng cơ sở sau GD1-12. |
| GiaoDichKho | PK `{MaGDKho}` | `MaGDKho → MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem`; BCNF. Nhật ký bất biến; `MaNV` hoặc `MaHD` có thể rỗng cho loại điều chỉnh không gắn người dùng/hóa đơn nếu nhóm cho phép. |

## Kết luận dạng chuẩn toàn lược đồ

Có 20 quan hệ: A 7, B 4, C 4, D 5. Theo tập PTH toàn quan hệ đã công bố, 19 quan hệ đạt BCNF; HoiVien đạt 2NF, không đạt 3NF vì TongChiTieu12T → Hang, trong đó TongChiTieu12T không là siêu khóa và Hang không là thuộc tính khóa. Do đó mức chuẩn chung bảo đảm của toàn lược đồ là **2NF, không tuyên bố toàn CSDL đạt 3NF/BCNF**.

Giữ Hang là quyết định lưu tổng/hạng đã chốt; bắt buộc đồng bộ theo B. Phương án chuẩn hóa nếu muốn đổi chính sách sau này: bỏ Hang khỏi bảng cơ sở, tính từ tổng khi đọc; bản hiện tại không tự đổi phương án người dùng đã chọn. TongTien/TonKho/TongChiTieu12T/NhanVien.ChucVu là tổng hoặc bản sao liên bảng cần ràng buộc nhất quán; sự dư thừa đó không tự tạo PTH ngoài khóa trong chính bảng nếu không có định thức nội bảng tương ứng.

Không đưa MaHV/MaGoi vào GoiHoiVien cơ sở: MaHD xác định MaHV và (MaHD,STT) xác định MaGoi, là phụ thuộc vào một phần AK (MaHD,STT,DonVi). Không lưu MaHV trên LichHenPT; chủ lịch suy ra qua MaGHV. Không lưu ThanhTien trên ChiTietHoaDon; tránh PTH (SoLuong,DonGia) → ThanhTien ngoài khóa. Không áp dụng kết luận chuẩn hóa của bảng cơ sở cho kết quả nối có thêm thuộc tính dẫn xuất.

Các ràng buộc chỉ áp dụng cho dòng chưa hủy, phiên đang mở, dòng loại gói/sản phẩm, giao dịch kho bán hàng là ràng buộc có điều kiện; không biến chúng thành AK/PTH trên toàn quan hệ. BCNF ở đây dựa trên tập PTH đã công bố, không phải bằng chứng các ràng buộc thời gian/tổng hợp tự được bảo đảm.

## Từ điển thuộc tính

Các mô tả đầy đủ được giữ tại module sở hữu, là thành phần của hồ sơ tổng:

| Module | Quan hệ | Từ điển và miền thuộc tính |
|---|---|---|
| A | 7 quan hệ chi nhánh/nhân viên/công tác/hoa hồng | [Ánh xạ và thuộc tính A](a/thiet-ke-logic.md) |
| B | 4 quan hệ hội viên/danh mục/gói đã mua/điểm | [Thuộc tính và miền B](b/thiet-ke-logic.md) |
| C | 4 quan hệ cửa/phiên vào ra/lịch PT/đánh giá | [Thông tin thuộc tính C](c/thiet-ke-logic.md) |
| D | 5 quan hệ sản phẩm/kho/hóa đơn/chi tiết/nhật ký | [Thông tin thuộc tính D](d/thiet-ke-logic.md) |

Miền chung: mã là định danh ổn định; các cột mã FK cùng miền với khóa cha. Tiền là số đồng nguyên, số lượng/buổi là số nguyên; dấu và giá trị rỗng theo miền từng bảng. Ngày DATE gồm hai biên; thời điểm dùng Asia/Saigon. Mọi PK/AK bắt buộc có giá trị; FK bắt buộc trừ các trường hợp tùy chọn nêu dưới đây.

## Danh mục khóa ngoại tổng

| Bảng con | Cột FK | Bảng cha / khóa | Bắt buộc |
|---|---|---|---|
| CungCapTienIch | MaCN | ChiNhanh.MaCN | Có |
| CungCapTienIch | MaTI | TienIch.MaTI | Có |
| LichSuCongTac | MaNV | NhanVien.MaNV | Có |
| LichSuCongTac | MaCN | ChiNhanh.MaCN | Có |
| KyHoaHong | MaCN | ChiNhanh.MaCN | Có |
| KyHoaHong | MaNVChot | NhanVien.MaNV | Có |
| ChiTietHoaHong | MaLichHen | LichHenPT.MaLichHen | Có |
| ChiTietHoaHong | MaCN, Thang | KyHoaHong.(MaCN,Thang) | Có |
| HoiVien | MaCNDangKy | ChiNhanh.MaCN | Có |
| GoiHoiVien | MaHD, STT | ChiTietHoaDon.(MaHD,STT) | Có |
| GhiNhanLoyalty | MaHD | HoaDon.MaHD | Có |
| CuaKiemSoat | MaCN | ChiNhanh.MaCN | Có |
| CheckIn | MaHV | HoiVien.MaHV | Có |
| CheckIn | MaCN | ChiNhanh.MaCN | Có |
| CheckIn | MaCN, SoHieuCuaVao | CuaKiemSoat.(MaCN,SoHieuCua) | Có |
| CheckIn | MaCN, SoHieuCuaRa | CuaKiemSoat.(MaCN,SoHieuCua) | Khi có cửa ra; cùng MaCN với cửa vào |
| LichHenPT | MaGHV | GoiHoiVien.MaGHV | Có |
| LichHenPT | MaNV | NhanVien.MaNV | Có |
| LichHenPT | MaCN | ChiNhanh.MaCN | Có |
| DanhGiaHLV | MaLichHen | LichHenPT.MaLichHen | Có; duy nhất |
| TonKho | MaCN | ChiNhanh.MaCN | Có |
| TonKho | MaSP | SanPham.MaSP | Có |
| HoaDon | MaHV | HoiVien.MaHV | Có |
| HoaDon | MaNV | NhanVien.MaNV | Có |
| HoaDon | MaCN | ChiNhanh.MaCN | Có |
| ChiTietHoaDon | MaHD | HoaDon.MaHD | Có |
| ChiTietHoaDon | MaGoi | GoiDichVu.MaGoi | Chỉ dòng gói |
| ChiTietHoaDon | MaSP | SanPham.MaSP | Chỉ dòng sản phẩm |
| GiaoDichKho | MaCN | ChiNhanh.MaCN | Có |
| GiaoDichKho | MaSP | SanPham.MaSP | Có |
| GiaoDichKho | MaNV | NhanVien.MaNV | Có thể rỗng với ghi nhận không gắn người dùng |
| GiaoDichKho | MaHD | HoaDon.MaHD | Bắt buộc khi xuất bán; loại khác có thể rỗng |
| GiaoDichKho | MaCN, MaSP | TonKho.(MaCN,MaSP) | Có |

Tổng 33 FK, gồm 5 FK ghép. FK chỉ bảo đảm tồn tại, không thay thế điều kiện loại dòng, chủ gói, đủ quyền lợi hay thời gian. Không xóa dây chuyền lịch sử mua, tập, kho, điểm hoặc kỳ đã chốt.

## Hợp đồng dữ liệu giữa module

| Nơi cung cấp → nơi dùng | Dữ liệu / yêu cầu thống nhất |
|---|---|
| A → B/C/D | ChiNhanh, NhanVien dùng cùng mã; C đọc LichSuCongTac.ChucVu/MaCN theo toàn khoảng buổi hẹn; D đọc sức chứa/giờ cho báo cáo |
| D → B | HoaDon.MaHV/NgayLap/TongTien; ChiTietHoaDon.MaGoi/LoaiDong/SoLuong; tạo mỗi đơn vị đúng một MaGHV |
| B → C | vGoiHoiVien = GoiHoiVien nối ChiTietHoaDon theo (MaHD,STT), nối HoaDon theo MaHD; lấy MaHV từ HoaDon, MaGoi từ dòng. Đây là phép nối logic, không phải yêu cầu cài View |
| C → A | LichHenPT.MaLichHen/MaNV/MaCN/NgayGio/TrangThai; chỉ buổi hoàn thành hợp lệ vào danh sách chốt; bảo vệ nguồn kỳ cũ |
| C → D | CheckIn.MaCN/TGVao/TGRa/SoHieuCuaRa; số có mặt khác tổng lượt vào và khác tải yêu cầu đồng thời |
| D ↔ B/C | Hóa đơn/gói/điểm/kho cùng thành công hoặc hoàn tác; quyền lợi chỉ sử dụng sau thanh toán thành công, không đọc quyền lợi chưa hoàn tất |

## Ràng buộc liên module và bảng tầm ảnh hưởng

Dấu +: thao tác có thể làm sai mệnh đề, phải kiểm tra; −: không trực tiếp làm sai mệnh đề đó. Thứ tự luôn Thêm / Xóa / Sửa; cột sửa nêu trường liên quan. Quy tắc bất biến/lịch sử là ràng buộc riêng: dấu − của một mệnh đề không có nghĩa được phép xóa. Với quy tắc kiểm tra tại thời điểm nghiệp vụ, thay đổi danh mục về sau không làm giao dịch quá khứ tự trở thành sai.

| ID | Bối cảnh | Nội dung | Tầm ảnh hưởng (Thêm / Xóa / Sửa) |
|---|---|---|---|
| T-R01 | Toàn bộ 33 FK | Bản ghi con tham chiếu đúng khóa cha, đúng thứ tự/cặp cột; không có dữ liệu mồ côi | Mỗi bảng con (+/−/+ cột FK); mỗi bảng cha (−/+/+ khóa được tham chiếu). Danh mục cụ thể ở bảng FK tổng |
| T-R02 | HoaDon, ChiTietHoaDon, GoiHoiVien, GoiDichVu | Hóa đơn hoàn tất có ít nhất một dòng. Dòng gói có đúng n đơn vị với DonVi=1..n; dòng sản phẩm có 0 gói. Mỗi gói đúng chủ hóa đơn, đúng danh mục dòng; chủ/loại không lưu lặp | HoaDon (+/+/+ MaHD,MaHV); ChiTietHoaDon (+/+/+ MaHD,STT,LoaiDong,MaGoi,MaSP,SoLuong); GoiHoiVien (+/+/+ MaGHV,MaHD,STT,DonVi); GoiDichVu (−/+/+ MaGoi,LoaiGoi,CachTinh) |
| T-R03 | Thanh toán D và quyền lợi/điểm B, kho D | Hóa đơn/dòng/gói/ghi nhận điểm/tổng hội viên/nhật ký và số dư kho cùng thành công hoặc hoàn tác. Lặp MaHD không tạo lại, không trừ/cộng lần hai; không sửa hóa đơn đã trả tiền | HoaDon/ChiTietHoaDon/GoiHoiVien/GhiNhanLoyalty/GiaoDichKho (+/+/+ dữ liệu giao dịch); HoiVien (−/+/+ tổng,điểm,hạng); TonKho (−/+/+ số dư). Tính nguyên tử là yêu cầu chuyển trạng thái toàn bộ, không chỉ một dấu kiểm tra bảng |
| T-R04 | GoiHoiVien, HoaDon, ChiTietHoaDon, GoiDichVu | Gym/Yoga theo hạn cùng hội viên/loại nối tiếp sau gói cuối cùng, gồm gói chờ; không chồng. PT mua thêm có hạn/buổi riêng và dùng ngay; vé lượt không nối hạn. Mua PT cần quyền vào hiện có hoặc mua kèm, chỉ kiểm tra tại lần mua | GoiHoiVien (+/−/+ ngày,số dư,MaHD,STT); HoaDon/ChiTietHoaDon (−/+/+ chủ,liên kết,loại,mã gói); GoiDichVu (−/+/+ loại,cách tính,hạn,số buổi). Việc tạo dòng cùng hóa đơn cũng thuộc T-R02/03 |
| T-R05 | CheckIn, CuaKiemSoat, quyền vào B | Tại lúc vào: cửa đúng chi nhánh/chiều, đang hoạt động; có thẻ theo hạn hợp lệ hoặc vé còn lượt; PT không mở cổng. Trừ vé và tạo phiên cùng thành công; quét lặp phiên mở không trừ thêm. Tối đa một phiên mở/hội viên. Sau ra, vào lại phải còn quyền hợp lệ | CheckIn (+/−/+ MaHV,MaCN,cửa,TGVao,TGRa); dữ liệu ứng viên tại lúc vào: GoiHoiVien (+/+/+ ngày,số dư,liên kết), GoiDichVu (+/+/+ loại,cách tính), HoaDon/ChiTietHoaDon (+/+/+ chủ,liên kết,loại); CuaKiemSoat (+/+/+ khóa,chiều,trạng thái). Không dùng số dư hiện nay để xét lại lượt vào quá khứ |
| T-R06 | LichHenPT, GoiHoiVien, hóa đơn/dòng, danh mục | Đặt đúng gói PT của hội viên, hạn bao phủ ngày hẹn. Số lịch Đã đặt/chưa trừ kể cả quá giờ <= SoBuoiConLai. Đến giờ trừ đúng MaGHV một lần, giảm số dư và bật cờ cùng nhau. Hủy còn >=2 giờ giải phóng giữ chỗ; vắng/hoàn thành không trừ lần hai | LichHenPT (+/−/+ MaGHV,NgayGio,TrangThai,DaTruBuoi,LyDoKetThuc); GoiHoiVien (−/+/+ ngày,số dư,liên kết); HoaDon/ChiTietHoaDon/GoiDichVu (−/+/+ chủ,loại,liên kết). Tới NgayGio kích hoạt nghĩa vụ trừ, không cần CheckIn |
| T-R07 | LichHenPT, LichSuCongTac, NhanVien | QD10: khung một giờ bắt đầu giờ tròn, cùng PT không có hai lịch chưa hủy cùng NgayGio. Toàn buổi có chức vụ PT tại chi nhánh lịch theo lịch sử. Đổi vị trí từ ngày mai bị từ chối nếu làm lịch chưa hủy không phù hợp; không tự chuyển/hủy lịch | LichHenPT (+/−/+ MaNV,MaCN,NgayGio,TrangThai); LichSuCongTac (+/+/+ MaNV,MaCN,ChucVu,ngày). NhanVien.ChucVu hiện tại không thay thế lịch sử; sửa/xóa khóa nhân viên chịu T-R01 |
| T-R08 | NhanVien và LichSuCongTac | Nhân viên đang làm có đúng một giai đoạn hôm nay; ChucVu hiện tại khớp lịch sử. Kết thúc ngày D, giai đoạn mới từ D+1; một thay đổi đang chờ/nhân viên; không chồng ngày | NhanVien (+/−/+ ChucVu,TrangThaiLamViec); LichSuCongTac (+/+/+ MaNV,ChucVu,MaCN,ngày). Sang ngày mới phải đồng bộ, không lấy NgayKetThuc rỗng làm điều kiện hiện hành duy nhất |
| T-R09 | KyHoaHong, ChiTietHoaHong, LichHenPT, LichSuCongTac, ChiNhanh | Kỳ tháng đã kết thúc chứa đủ buổi hoàn thành hợp lệ tại chi nhánh; mỗi buổi một lần; vắng/hủy không có hoa hồng. Chụp mức khi chốt và danh sách buổi, bảo vệ kỳ và nguồn, không nhận thêm buổi sau chốt | KyHoaHong (+/+/+ cột kỳ/mức/người/thời điểm); ChiTietHoaHong (+/+/+); LichHenPT (+/+/+ MaNV,MaCN,NgayGio,TrangThai); LichSuCongTac (+/+/+ cột giai đoạn). ChiNhanh.HoaHongMoiBuoi chỉ ảnh hưởng chọn mức khi chốt kỳ mới, không ảnh hưởng tính lại kỳ cũ |
| T-R10 | HoaDon, ChiTietHoaDon và báo cáo D | Tổng sau giảm = tổng SoLuong×DonGia − GiamGia. Phân bổ giảm trên mọi dòng theo tỷ lệ, chia đồng dư theo phần dư giảm dần/STT tăng; tổng giảm dòng = GiamGia, tổng doanh thu dòng = TongTien. Phân bổ trước khi lọc báo cáo | HoaDon (+/−/+ GiamGia,TongTien); ChiTietHoaDon (+/+/+ MaHD,STT,SoLuong,DonGia). Giá danh mục mới không đổi đơn giá lịch sử. Xóa hóa đơn bị cấm riêng bởi T-R03 |
| T-R11 | HoaDon, GhiNhanLoyalty, HoiVien | Một ghi nhận/hóa đơn; điểm=floor(TongTien/100.000), tích lũy không hết hạn. Tổng xét hạng gồm các khoản NgayLap<=T<NgayLap+12 tháng (kẹp ngày cuối tháng). Trước đọc/dùng hạng xử lý khoản đã hết hạn, hạ hạng nếu cần, không trừ điểm | HoaDon (+/+/+ MaHV,TongTien,NgayLap); GhiNhanLoyalty (+/+/+); HoiVien (+/−/+ tổng,điểm,hạng); thời gian đến mốc hết hạn làm phát sinh đồng bộ. Xóa hội viên còn hóa đơn bị T-R01 chặn |
| T-R12 | TonKho, GiaoDichKho, HoaDon, ChiTietHoaDon, SanPham | Tổng tồn đúng nhật ký, không âm. Xuất bán đúng chi nhánh hóa đơn và lượng gộp theo sản phẩm trên tất cả dòng; số dư mở đầu cũng có nhật ký. Điều kiện đang bán/đủ tồn xét tại lúc bán | TonKho (+/+/+ khóa,số dư); GiaoDichKho (+/+/+ khóa kho,MaHD,loại,lượng); HoaDon (−/+/+ MaCN); ChiTietHoaDon (+/+/+ MaHD,MaSP,SoLuong,LoaiDong); SanPham (+/−/+ DangKinhDoanh) tại bước kiểm tra ứng viên bán, FK riêng theo T-R01 |
| T-R13 | CheckIn, ChiNhanh và báo cáo lấp đầy | N(T) đếm cùng chi nhánh có TGVao<=T và (TGRa rỗng hoặc TGRa>T). Tỷ lệ 100×N/SucChua; sức chứa dương, cho phép >100%, không chặn cổng. Báo cáo giờ chụp đầu khung. Tự đóng dùng mốc đóng cửa ngày vào, đánh dấu bằng cửa ra rỗng; báo cáo tạm tính nếu còn lượt quá hạn chưa đóng | CheckIn (+/+/+ MaCN,TGVao,TGRa,SoHieuCuaRa); ChiNhanh (−/+/+ MaCN,SucChua,GioMoCua,GioDongCua). Thời gian đến giờ đóng phát sinh xử lý; giờ/sức chứa cố định trong kỳ mô phỏng |
| T-R14 | DanhGiaHLV, LichHenPT, gói và hóa đơn | Mỗi lịch hoàn thành có tối đa một đánh giá, điểm nguyên 1–5; người đánh giá là chủ hội viên suy qua gói→dòng→hóa đơn; không lưu lặp chủ/PT vào đánh giá | DanhGiaHLV (+/−/+ MaLichHen,Diem,ThoiDiemDanhGia); LichHenPT (−/+/+ MaGHV,TrangThai,NgayGio); GoiHoiVien/HoaDon/ChiTietHoaDon (−/+/+ liên kết,chủ). Bất biến nguồn sau thanh toán áp dụng cùng T-R03 |

Miền, khóa, ràng buộc nội module và dấu ảnh hưởng chi tiết vẫn ở A-R01–11, B-R01–09, C-R01–13, D-R01–08. Bảng tổng bổ sung các bảng nguồn qua phép nối, không thay thế quy tắc cấm sửa/xóa hoặc thu hẹp ràng buộc nội module.

## Đối chiếu chuỗi nghiệp vụ

| Điểm nối | Kết quả cần giữ |
|---|---|
| Mua 2 đơn vị PT trên một dòng | Hai MaGHV, DonVi=1/2, cùng FK dòng; lịch chọn đúng một gói, không chuyển buổi giữa hai gói |
| Mua Gym còn hạn tới 14/11 | Gói nối bắt đầu 15/11; nếu đã có gói chờ phải nối sau đuôi chuỗi đó |
| Mua PT kèm vé lượt | Thanh toán tạo cả quyền lợi; vé dùng khi vào cổng, PT dùng khi tới giờ hẹn; không trừ chéo |
| PT 18h, vào 8h rồi hủy 12h | Cổng tiêu thụ quyền vào riêng; hủy hợp lệ giải phóng giữ PT, không trừ PT |
| PT 18h, hủy 16h / 16h01 | 16h được hủy; 16h01 bị từ chối. Tới 18h lịch chưa hủy trừ một buổi; chạy lại không trừ thêm |
| Cùng PT: 17h–18h và 18h–19h | Hai khung liền nhau hợp lệ; hai lịch chưa hủy cùng 17h bị chặn; 17h30 bị từ chối theo QD10 |
| Đổi vị trí ngày 15 | Hiệu lực ngày 16; nếu lịch chưa hủy sau mốc đó không phù hợp thì từ chối toàn bộ thay đổi |
| Buổi hoàn thành / không đến | Chỉ hoàn thành có hoa hồng và được đánh giá; cả hai đã trừ PT tại giờ hẹn |
| Sửa mức hoa hồng sau chốt | Kỳ cũ dùng mức và danh sách cũ, không tính lại từ cấu hình mới |
| Hóa đơn vừa hết cửa sổ 12 tháng | Loại tiền khỏi tổng xét hạng đúng một lần, giữ điểm và hóa đơn; hạng được đồng bộ trước khi dùng |
| Vào đúng T / ra đúng T | Vào đúng T có tính hiện diện, ra đúng T không tính; sức chứa chỉ dùng báo cáo |
| Chạy lại thanh toán cùng MaHD | Trả kết quả đã có; không thêm gói/điểm hoặc trừ tồn lần hai |

Đây là đối chiếu yêu cầu và mô hình dữ liệu GĐ1, không phải kết quả kiểm thử ứng dụng hay SQL Server.

## Giới hạn và thông tin cần bổ sung khi nộp

- Ngưỡng hạng 20/50 triệu là giả định nhóm đang dùng, cần giảng viên xác nhận; không ghi như ngưỡng do đề cho. QD10 đã được người dùng chốt giờ tròn, không còn là câu hỏi mở.
- Bảo lưu chưa chốt và chưa đưa vào luồng/lược đồ; không tự thêm chức năng bảo lưu. Hoàn tiền/đổi điểm không thuộc phương án hiện tại. Giờ hoạt động trong ngày và sức chứa cố định là quy ước mô phỏng QD15.
- Số liệu tần suất là giả định chung GD1-15/16; yêu cầu đồng thời và mục tiêu <50 ms chưa được kiểm chứng. Nền tảng ứng dụng, index, partition, thủ tục, ma trận truy vấn và benchmark dành cho GĐ2.
- README còn thiếu tên/MSSV của một số thành viên; không tự điền danh tính. Các ô này cần người trong nhóm bổ sung trước khi nộp, không ảnh hưởng cấu trúc CSDL.
