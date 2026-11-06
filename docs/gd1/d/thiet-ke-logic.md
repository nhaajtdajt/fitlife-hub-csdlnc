# Module D — Thiết kế mức logic

Các kiểu SQL cụ thể (độ dài mã, `DECIMAL`, `DATETIME2`, collation) sẽ được chốt khi viết script ở giai đoạn 2. Dưới đây dùng khóa chung `MaCN`, `MaNV`, `MaHV`, `MaGoi`, `MaHD` đúng hợp đồng. Thực thể của module khác chỉ ghi các cột D tham chiếu.

## Lược đồ quan hệ

```text
SanPham(MaSP, TenSP, LoaiSP, GiaBan, DangKinhDoanh)
TonKho(MaCN, MaSP, SoLuongTon)
HoaDon(MaHD, MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan)
ChiTietHoaDon(MaHD, STT, LoaiDong, MaGoi, MaSP, SoLuong, DonGia, ThanhTien)
GiaoDichKho(MaGDKho, MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem)
```

`ChiNhanh(MaCN, SucChua)`, `NhanVien(MaNV)`, `HoiVien(MaHV)` và `GoiDichVu(MaGoi, LoaiGoi, Gia)` là quan hệ ngoài module, được nêu để xác định khóa ngoại.

## Khóa, phụ thuộc hàm và dạng chuẩn

| Quan hệ | Khóa chính / khóa ứng viên | Phụ thuộc hàm chính | Dạng chuẩn và ghi chú |
|---|---|---|---|
| `SanPham` | PK `{MaSP}` | `MaSP → TenSP, LoaiSP, GiaBan, DangKinhDoanh` | BCNF: vế trái của mọi PTH không tầm thường là khóa. Tên sản phẩm không mặc định duy nhất. |
| `TonKho` | PK `{MaCN, MaSP}` | `(MaCN, MaSP) → SoLuongTon` | BCNF. Số dư tách riêng cho từng chi nhánh và mặt hàng. |
| `HoaDon` | PK `{MaHD}` | `MaHD → MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan` | BCNF. `TongTien` là tổng chụp tại thời điểm bán, được kiểm tra với chi tiết; không suy ra bằng PTH đơn trong cùng quan hệ. |
| `ChiTietHoaDon` | PK `{MaHD, STT}` | `(MaHD, STT) → LoaiDong, MaGoi, MaSP, SoLuong, DonGia, ThanhTien` | BCNF. Không lưu tên sản phẩm/gói vì phụ thuộc vào mã danh mục; `ThanhTien` được giữ như giá trị dòng đã chốt và phải bằng số lượng × đơn giá. |
| `GiaoDichKho` | PK `{MaGDKho}` | `MaGDKho → MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem` | BCNF. Nhật ký bất biến; `MaNV` hoặc `MaHD` có thể rỗng cho loại điều chỉnh không gắn người dùng/hóa đơn nếu nhóm cho phép. |

Các khóa ngoại:

- `TonKho.MaCN → ChiNhanh.MaCN`; `TonKho.MaSP → SanPham.MaSP`.
- `HoaDon.MaHV → HoiVien.MaHV`; `HoaDon.MaNV → NhanVien.MaNV`; `HoaDon.MaCN → ChiNhanh.MaCN`.
- `ChiTietHoaDon.MaHD → HoaDon.MaHD`; `MaGoi → GoiDichVu.MaGoi`; `MaSP → SanPham.MaSP` (hai FK tùy chọn, áp dụng điều kiện loại dòng).
- `GiaoDichKho.MaCN → ChiNhanh.MaCN`; `MaSP → SanPham.MaSP`; `MaNV → NhanVien.MaNV`; `MaHD → HoaDon.MaHD`.

## Thông tin thuộc tính

| Quan hệ | Thuộc tính | Ý nghĩa / miền giá trị |
|---|---|---|
| `SanPham` | `MaSP` | Mã sản phẩm duy nhất. |
| | `TenSP` | Tên hiển thị, bắt buộc. |
| | `LoaiSP` | Nhóm hàng, ví dụ whey protein, bình nước, thảm yoga. |
| | `GiaBan` | Giá hiện hành, lớn hơn 0; không dùng để tính lại hóa đơn cũ. |
| | `DangKinhDoanh` | Cờ cho phép bán; ngừng bán không xóa lịch sử. |
| `TonKho` | `MaCN`, `MaSP` | Khóa ghép xác định một mặt hàng tại một chi nhánh. |
| | `SoLuongTon` | Số nguyên không âm đang có thể bán. |
| `HoaDon` | `MaHD` | Mã hóa đơn duy nhất, công bố cho B. |
| | `MaHV`, `MaNV`, `MaCN` | Hội viên mua, nhân viên lập, chi nhánh phát sinh. |
| | `NgayLap` | Ngày giờ xác nhận giao dịch; công bố cho B. |
| | `GiamGia` | Tổng giảm giá tiền, không âm và không vượt tổng dòng. |
| | `TongTien` | Số phải trả sau giảm giá; công bố cho B để tính điểm/hạng. |
| | `HinhThucThanhToan` | Một trong Tiền mặt, VNPay, Thẻ. |
| `ChiTietHoaDon` | `MaHD`, `STT` | Hóa đơn và số thứ tự dòng; khóa ghép. |
| | `LoaiDong` | `GoiDichVu` hoặc `SanPham`. |
| | `MaGoi`, `MaSP` | Chỉ một cột có giá trị phù hợp `LoaiDong`. |
| | `SoLuong` | Số nguyên dương; gói thường có lượng 1. |
| | `DonGia` | Đơn giá chụp tại thời điểm thanh toán. |
| | `ThanhTien` | Thành tiền dòng đã chốt, bằng `SoLuong × DonGia`. |
| `GiaoDichKho` | `MaGDKho` | Mã nhật ký điều chỉnh tồn duy nhất. |
| | `MaCN`, `MaSP` | Nơi và mặt hàng bị thay đổi tồn. |
| | `MaNV`, `MaHD` | Nhân viên ghi nhận và hóa đơn liên quan; `MaHD` rỗng với nhập/kiểm kê. |
| | `LoaiGiaoDich` | Nhập, xuất bán hoặc điều chỉnh kiểm kê. |
| | `SoLuongThayDoi` | Số nguyên có dấu: nhập dương, xuất âm; không bằng 0. |
| | `ThoiDiem` | Thời điểm phát sinh nhật ký. |

## Kết luận chuẩn hóa

Mỗi quan hệ trên đạt BCNF theo các phụ thuộc hàm đã nêu, do mọi định thức không tầm thường là một khóa ứng viên. Dữ liệu danh mục không lặp trong dòng hóa đơn; số dư tồn và nhật ký thay đổi được tách riêng; đầu hóa đơn và các dòng được tách theo quan hệ 1–n. `HoaDon.TongTien` và `TonKho.SoLuongTon` là giá trị tổng hợp/dư thừa có chủ đích để phục vụ nghiệp vụ và hiệu năng, không phải lý do gộp các quan hệ; chúng cần được duy trì bằng giao dịch và ràng buộc nhất quán. Kết luận này áp dụng cho lược đồ D; kết luận toàn CSDL cần cả nhóm ghép lược đồ A–D.

## Ràng buộc logic

Ký hiệu `+`: thao tác có thể làm vi phạm; `−`: thao tác không thể làm vi phạm trực tiếp. Những ràng buộc tổng hợp nhiều dòng cần thủ tục/giao dịch hoặc trigger ở giai đoạn 2, không chỉ CHECK đơn dòng.

| ID | Bối cảnh | Nội dung | Bảng tầm ảnh hưởng (Thêm / Xóa / Sửa) |
|---|---|---|---|
| D-R01 | `ChiTietHoaDon` | Mỗi dòng có đúng một mã mặt hàng; `LoaiDong=GoiDichVu` thì `MaGoi` có, `MaSP` rỗng; `LoaiDong=SanPham` thì ngược lại. | `ChiTietHoaDon` (+ / + / + `LoaiDong, MaGoi, MaSP`) |
| D-R02 | `ChiTietHoaDon`, `HoaDon` | `SoLuong > 0`, `DonGia >= 0`, `ThanhTien = SoLuong × DonGia`; tổng `TongTien = SUM(ThanhTien) - GiamGia`, giảm giá nằm trong `[0, SUM(ThanhTien)]`. | `ChiTietHoaDon` (+ / + / + các cột tính tiền); `HoaDon` (+ / − / + `GiamGia, TongTien`) |
| D-R03 | `TonKho`, `GiaoDichKho` | Tồn không âm và bằng tổng số dư mở đầu cùng các thay đổi kho; giao dịch bán trừ đúng số lượng đã ghi trên dòng sản phẩm của hóa đơn. | `TonKho` (+ / − / + `SoLuongTon`); `GiaoDichKho` (+ / − / + `SoLuongThayDoi`); `ChiTietHoaDon` (+ / − / + `SoLuong, MaSP`) |
| D-R04 | `HoaDon`, `ChiTietHoaDon`, `TonKho` | Chỉ xác nhận bán khi mỗi sản phẩm đang kinh doanh và tồn tại đủ tại đúng `HoaDon.MaCN`; ghi hóa đơn, chi tiết, nhật ký kho và cập nhật số dư nguyên tử. | `HoaDon` (+ / − / + `MaCN`); `ChiTietHoaDon` (+ / − / + `MaSP, SoLuong`); `TonKho` (− / − / + `SoLuongTon`); `SanPham` (− / − / + `DangKinhDoanh`) |
| D-R05 | `HoaDon`, `ChiTietHoaDon` | Hóa đơn đã thanh toán có ít nhất một dòng; sau thanh toán không sửa/xóa hóa đơn hoặc dòng. | `HoaDon` (+ / + / +); `ChiTietHoaDon` (+ / + / +) — thao tác chỉ hợp lệ trước xác nhận thanh toán |
| D-R06 | `HoaDon` | Phương thức thanh toán nằm trong tập cho phép; `GiamGia >= 0`, `TongTien >= 0`; `TongTien` sau giảm giá được gửi đúng một lần cho B. | `HoaDon` (+ / − / + `HinhThucThanhToan, GiamGia, TongTien`) |
| D-R07 | `GiaoDichKho` | Số thay đổi khác 0; loại nhập có lượng dương, xuất bán có lượng âm; nhật ký đã ghi không bị xóa/sửa tùy tiện. | `GiaoDichKho` (+ / + / + `LoaiGiaoDich, SoLuongThayDoi`) |
| D-R08 | Báo cáo lấp đầy (`CheckIn`, `ChiNhanh`) | Số lượt đang ở chi nhánh tại thời điểm chụp không vượt `SucChua`; giờ vào/ra và quy tắc tự đóng lượt phải nhất quán. | `CheckIn` (C sở hữu: + / + / + `MaCN, TGVao, TGRa`); `ChiNhanh` (A sở hữu: − / − / + `SucChua`) |

## Điểm ghép với A/B/C

- A xác nhận `ChiNhanh.MaCN`, `ChiNhanh.SucChua`, `NhanVien.MaNV` đúng kiểu/khóa; D dùng `MaCN` trên hóa đơn để xác định doanh thu và kho.
- B xác nhận thủ tục `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)` và giao dịch tạo `GoiHoiVien` cùng quy trình thanh toán gói. Chữ ký đã có trong hợp đồng; chi tiết rollback liên module cần thống nhất trước triển khai.
- C xác nhận các cột nguồn check-in và cách đóng lượt quên quét ra để báo cáo lấp đầy nhất quán.

