# Module D — Thiết kế mức logic

Các kiểu SQL cụ thể (độ dài mã, `DECIMAL`, `DATETIME2`, collation) sẽ được chốt khi viết script ở giai đoạn 2. Dưới đây dùng khóa chung `MaCN`, `MaNV`, `MaHV`, `MaGoi`, `MaHD` đúng hợp đồng. Thực thể của module khác chỉ ghi các cột D tham chiếu.

## Lược đồ quan hệ

```text
SanPham(MaSP, TenSP, LoaiSP, GiaBan, DangKinhDoanh)
TonKho(MaCN, MaSP, SoLuongTon)
HoaDon(MaHD, MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan)
ChiTietHoaDon(MaHD, STT, LoaiDong, MaGoi, MaSP, SoLuong, DonGia)
GiaoDichKho(MaGDKho, MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem)
```

`ChiNhanh(MaCN, SucChua, GioMoCua, GioDongCua)`, `NhanVien(MaNV)`, `HoiVien(MaHV)` và `GoiDichVu(MaGoi, LoaiGoi, Gia)` là quan hệ ngoài module, được nêu để xác định khóa ngoại.

Báo cáo còn đọc CheckIn(MaHV, MaCN, TGVao, TGRa, SoHieuCuaRa) của C theo hợp đồng, không sở hữu hoặc ghi bảng này.

## Khóa, phụ thuộc hàm và dạng chuẩn

| Quan hệ | Khóa chính / khóa ứng viên | Phụ thuộc hàm chính | Dạng chuẩn và ghi chú |
|---|---|---|---|
| `SanPham` | PK `{MaSP}` | `MaSP → TenSP, LoaiSP, GiaBan, DangKinhDoanh` | BCNF: vế trái của mọi PTH không tầm thường là khóa. Tên sản phẩm không mặc định duy nhất. |
| `TonKho` | PK `{MaCN, MaSP}` | `(MaCN, MaSP) → SoLuongTon` | BCNF. Số dư tách riêng cho từng chi nhánh và mặt hàng. |
| `HoaDon` | PK `{MaHD}` | `MaHD → MaHV, MaNV, MaCN, NgayLap, GiamGia, TongTien, HinhThucThanhToan` | BCNF. `TongTien` là tổng chụp tại thời điểm bán, được kiểm tra với chi tiết; không suy ra bằng PTH đơn trong cùng quan hệ. |
| `ChiTietHoaDon` | PK `{MaHD, STT}` | `(MaHD, STT) → LoaiDong, MaGoi, MaSP, SoLuong, DonGia` | BCNF theo PTH đã nêu. Không lưu tên sản phẩm/gói hoặc ThanhTien dẫn xuất. (SoLuong, DonGia) → ThanhTien chỉ thuộc kết quả truy vấn mở rộng, không phải PTH của bảng cơ sở sau GD1-12. |
| `GiaoDichKho` | PK `{MaGDKho}` | `MaGDKho → MaCN, MaSP, MaNV, MaHD, LoaiGiaoDich, SoLuongThayDoi, ThoiDiem` | BCNF. Nhật ký bất biến; `MaNV` hoặc `MaHD` có thể rỗng cho loại điều chỉnh không gắn người dùng/hóa đơn nếu nhóm cho phép. |

Các khóa ngoại:

- `TonKho.MaCN → ChiNhanh.MaCN`; `TonKho.MaSP → SanPham.MaSP`.
- `HoaDon.MaHV → HoiVien.MaHV`; `HoaDon.MaNV → NhanVien.MaNV`; `HoaDon.MaCN → ChiNhanh.MaCN`.
- `ChiTietHoaDon.MaHD → HoaDon.MaHD`; `MaGoi → GoiDichVu.MaGoi`; `MaSP → SanPham.MaSP` (hai FK tùy chọn, áp dụng điều kiện loại dòng).
- `GiaoDichKho.MaCN → ChiNhanh.MaCN`; `MaSP → SanPham.MaSP`; `MaNV → NhanVien.MaNV`; `MaHD → HoaDon.MaHD`.
- `(GiaoDichKho.MaCN, GiaoDichKho.MaSP) → TonKho(MaCN, MaSP)`; nhật ký chỉ phát sinh cho cặp kho đã mở.

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
| | `GiamGia` | Tổng giảm giá hóa đơn bằng đồng nguyên, không âm và không vượt tổng dòng; được phân bổ theo quy tắc GD1-08 bên dưới. |
| | `TongTien` | Số phải trả sau giảm giá; công bố cho B để tính điểm/hạng. |
| | `HinhThucThanhToan` | Một trong Tiền mặt, VNPay, Thẻ. |
| `ChiTietHoaDon` | `MaHD`, `STT` | Hóa đơn và số thứ tự dòng; khóa ghép. |
| | `LoaiDong` | `GoiDichVu` hoặc `SanPham`. |
| | `MaGoi`, `MaSP` | Chỉ một cột có giá trị phù hợp `LoaiDong`. |
| | `SoLuong` | Số nguyên dương; gói thường có lượng 1. |
| | `DonGia` | Đơn giá chụp tại thời điểm thanh toán, số đồng nguyên không âm; dùng số chính xác, không FLOAT. |
| `GiaoDichKho` | `MaGDKho` | Mã nhật ký điều chỉnh tồn duy nhất. |
| | `MaCN`, `MaSP` | Nơi và mặt hàng bị thay đổi tồn. |
| | `MaNV`, `MaHD` | Nhân viên ghi nhận và hóa đơn liên quan; `MaHD` rỗng với nhập/kiểm kê. |
| | `LoaiGiaoDich` | Nhập, xuất bán hoặc điều chỉnh kiểm kê. |
| | `SoLuongThayDoi` | Số nguyên có dấu: nhập dương, xuất âm; không bằng 0. |
| | `ThoiDiem` | Thời điểm phát sinh nhật ký. |

## Phân bổ giảm giá và doanh thu dòng (GD1-08)

Đây là quy tắc cho các giá trị tính ra khi hiển thị hóa đơn/báo cáo, không thêm cột lưu GiamGiaDong hoặc DoanhThuDong. Đầu vào là dữ liệu hóa đơn đã thanh toán bất biến, đơn giá chụp lúc bán; không đọc giá danh mục hiện tại. Dùng chung một phép tính cho màn hình và báo cáo, triển khai truy vấn/View ở GĐ2.

Với mỗi hóa đơn, đặt A_i = SoLuong_i × DonGia_i (ThanhTien trước giảm), S = tổng A_i của **toàn bộ** dòng, G = HoaDon.GiamGia. Quy ước GĐ1 dùng tiền Việt Nam theo đồng nguyên, số lượng nguyên dương, đơn giá không âm; G phải là số nguyên thỏa 0 <= G <= S. Nếu đầu vào hoặc TongTien không khớp S − G, báo dữ liệu không hợp lệ, không âm thầm sửa để báo cáo khớp.

- S = 0: chỉ chấp nhận G = 0, mọi giảm giá dòng/doanh thu dòng đều bằng 0; không chia cho 0.
- S > 0: tính B_i = phần nguyên của (G × A_i) / S, P_i = phần dư của (G × A_i) khi chia S.
- R = G − tổng B_i. Cộng thêm 1 đồng vào B_i của R dòng có P_i lớn nhất; nếu bằng phần dư thì ưu tiên STT tăng dần. Khi R = 0 thì không cộng thêm. Dòng A_i = 0 không được phân bổ giảm giá.
- GiamGiaDong_i là B_i sau bước trên; DoanhThuDong_i = A_i − GiamGiaDong_i. Luôn bảo đảm 0 <= GiamGiaDong_i <= A_i, tổng GiamGiaDong = G và tổng DoanhThuDong = S − G = TongTien. Giảm 100% thì doanh thu tất cả dòng bằng 0.
- Dùng số nguyên/DECIMAL chính xác với độ rộng đủ cho tích G × A_i; giới hạn miền tiền và kiểm tra tràn được xác định ở thiết kế vật lý. Không dùng FLOAT hoặc làm tròn độc lập từng dòng rồi bỏ phần chênh.

Phân bổ trên **toàn hóa đơn trước**, rồi mới lọc MaGoi/MaSP/LoaiDong để nhóm báo cáo. Dòng có nhiều đơn vị lấy cả SoLuong × DonGia làm trọng số. Tổng doanh thu gói + sản phẩm bằng tổng hóa đơn trong cùng khoảng ngày/chi nhánh, không trừ GiamGia lần nữa sau khi cộng doanh thu dòng. B vẫn nhận TongTien đúng một lần để tính điểm/hạng trên hóa đơn, không tính điểm riêng từng dòng rồi cộng.

| Tình huống | Giảm giá phân bổ | Doanh thu sau giảm |
|---|---|---|
| Gym 1.000.000đ, Yoga 500.000đ; giảm 150.000đ | 100.000đ; 50.000đ | 900.000đ; 450.000đ; tổng 1.350.000đ |
| Gói 1.000.000đ, sản phẩm 500.000đ; giảm 150.000đ | Vẫn 100.000đ; 50.000đ trước khi lọc loại | Gói 900.000đ; sản phẩm 450.000đ |
| Ba dòng 1đ, giảm 2đ; STT 1, 2, 3 | 1đ; 1đ; 0đ (cùng phần dư, ưu tiên STT) | 0đ; 0đ; 1đ |
| Dòng 0đ và dòng 100đ; giảm 10đ | 0đ; 10đ | 0đ; 90đ |
| Hóa đơn một dòng / không giảm / giảm toàn bộ | Phân bổ toàn phần giảm cho dòng duy nhất / đều 0 / bằng giá trị dòng | Tổng luôn khớp TongTien |

GD1-12: ThanhTien là bí danh tính ra bằng SoLuong × DonGia khi đọc, không lưu trên bảng cơ sở. Màn hình/in hóa đơn/phân bổ giảm giá dùng cùng biểu thức này. Giá danh mục hiện tại không tham gia; đơn giá chụp lúc bán và hóa đơn đã thanh toán vẫn bất biến. Không áp dụng kết luận BCNF của bảng cơ sở cho quan hệ dẫn xuất đã thêm thuộc tính dẫn xuất.

## Kết luận chuẩn hóa

Mỗi quan hệ trên đạt BCNF theo các phụ thuộc hàm đã nêu, do mọi định thức không tầm thường là một khóa ứng viên. Dữ liệu danh mục không lặp trong dòng hóa đơn; số dư tồn và nhật ký thay đổi được tách riêng; đầu hóa đơn và các dòng được tách theo quan hệ 1–n. `HoaDon.TongTien` và `TonKho.SoLuongTon` là giá trị tổng hợp/dư thừa có chủ đích để phục vụ nghiệp vụ và hiệu năng, không phải lý do gộp các quan hệ; chúng cần được duy trì bằng giao dịch và ràng buộc nhất quán. Kết luận này áp dụng cho lược đồ D; [thiết kế tổng A–D](../thiet-ke-tong.md) kết luận toàn lược đồ bảo đảm 2NF vì ngoại lệ HoiVien.

## Ràng buộc logic

Ký hiệu `+`: thao tác đơn lẻ có thể làm vi phạm mệnh đề đang xét; `−`: không thể trực tiếp làm vi phạm mệnh đề đó. Đánh giá từ dữ liệu đang hợp lệ, trước khi tính các thao tác bù trong cùng giao dịch. Dấu − không cấp quyền xóa: FK, tính bất biến và ràng buộc khác vẫn có thể cấm. Với dữ liệu lịch sử, sửa/xóa là phép thử phân tích, không phải thao tác được cho phép.

| ID | Bối cảnh và nội dung | Tầm ảnh hưởng (Thêm / Xóa / Sửa) |
|---|---|---|
| D-R01 | Mỗi ChiTietHoaDon có đúng một mã hàng phù hợp LoaiDong | ChiTietHoaDon (+ / − / + LoaiDong,MaGoi,MaSP). Xóa dòng không làm dòng còn lại sai điều kiện này |
| D-R02 | SoLuong nguyên > 0, DonGia đồng nguyên >= 0; TongTien = SUM(SoLuong × DonGia) − GiamGia, 0 <= GiamGia <= tổng trước giảm. Phân bổ GD1-08 tính từ nguồn, không lưu ThanhTien | ChiTietHoaDon (+ / + / + MaHD,SoLuong,DonGia); HoaDon (+ / − / + MaHD,GiamGia,TongTien). Xóa đầu hóa đơn không vi phạm công thức của đầu hóa đơn còn tồn tại nhưng có thể vi phạm FK/D-R05 |
| D-R03 | Tồn không âm; với mỗi dòng TonKho, SoLuongTon bằng tổng nhật ký cùng (MaCN,MaSP), tổng rỗng là 0. Số dư ban đầu đã ghi thành nhật ký mở đầu, không cộng hai lần. Với mỗi (MaHD,MaSP) bán hàng, tổng số xuất phải khớp tổng SoLuong các dòng sản phẩm cùng mã tại đúng chi nhánh hóa đơn; không giả định mỗi sản phẩm chỉ xuất hiện một dòng | Đối với cân bằng tồn: TonKho (+ / − / + MaCN,MaSP,SoLuongTon); GiaoDichKho (+ / + / + MaCN,MaSP,SoLuongThayDoi). Đối với đối soát bán: GiaoDichKho (+ / + / + MaHD,MaCN,MaSP,LoaiGiaoDich,SoLuongThayDoi); ChiTietHoaDon (+ / + / + MaHD,LoaiDong,MaSP,SoLuong); HoaDon (− / + / + MaHD,MaCN). Xóa TonKho còn được kiểm tra FK/phụ thuộc ở dòng dưới |
| D-R04 | Tại lúc thanh toán, sản phẩm phải đang bán và có dòng tồn đúng chi nhánh, đủ số lượng gộp của tất cả dòng cùng sản phẩm. Hóa đơn, xuất kho, tạo gói/điểm phải commit/rollback cùng nhau. Đây là điều kiện lúc giao dịch, không bắt hóa đơn quá khứ sai khi sản phẩm ngừng bán về sau | Các thay đổi buộc kiểm tra lại ứng viên thanh toán: HoaDon (+ / − / + MaCN); ChiTietHoaDon (+ / − / + MaHD,LoaiDong,MaSP,SoLuong); TonKho (− / + / + MaCN,MaSP,SoLuongTon); SanPham (− / + / + MaSP,DangKinhDoanh). Nguyên tử kiểm tra qua D-R02/03/06 và hợp đồng B, không suy ra chỉ từ một dấu bảng |
| D-R05 | Hóa đơn đã thanh toán có ít nhất một dòng và bất biến. Không thêm dòng vào hóa đơn đã chốt, không sửa/xóa đầu hoặc chi tiết | Với số dòng tối thiểu: HoaDon (+ / − / + MaHD); ChiTietHoaDon (− / + / + MaHD). Với bất biến: HoaDon (− / + / + mọi cột); ChiTietHoaDon (+ / + / + mọi cột khi thuộc hóa đơn đã thanh toán). Tạo đầy đủ đầu/dòng được thực hiện trước cùng commit thanh toán |
| D-R06 | Miền HinhThucThanhToan đúng tập, GiamGia/TongTien không âm. Mỗi hóa đơn đã hoàn tất có đúng một GhiNhanLoyalty, đúng chủ/tiền/ngày; tổng/điểm/hạng B đồng bộ theo GD1-09 | Miền đầu hóa đơn: HoaDon (+ / − / + HinhThucThanhToan,GiamGia,TongTien). Ghi nhận/đồng bộ: HoaDon (+ / + / + MaHD,MaHV,TongTien,NgayLap); GhiNhanLoyalty (+ / + / + MaHD,DiemCong,DaLoaiKhoi12T); HoiVien (+ / + / + MaHV,TongChiTieu12T,DiemTichLuy,Hang). Chạy lại lời gọi cũng cần chống cộng lặp dù không thêm hóa đơn |
| D-R07 | Nhật ký có lượng nguyên khác 0; Nhập dương, Xuất bán âm, Điều chỉnh có thể dương/âm. Nhật ký đã ghi là bất biến, điều chỉnh tồn bằng dòng mới | Miền giá trị: GiaoDichKho (+ / − / + LoaiGiaoDich,SoLuongThayDoi). Bất biến: GiaoDichKho (− / + / + mọi cột đã ghi). Xóa không vi phạm miền của dòng còn lại nhưng có thể phá cân bằng D-R03 và bị cấm bởi bất biến |
| D-R08 | SucChua nguyên dương phục vụ công thức báo cáo lấp đầy. Không chặn cổng, không giới hạn tỷ lệ ở 100% | ChiNhanh (+ / − / + SucChua); CheckIn (− / − / − đối với miền SucChua). Sửa/xóa/thêm CheckIn thay đổi kết quả báo cáo, không tự tạo vi phạm miền sức chứa. FK và thời gian vào/ra được kiểm tra ở module C |

### Khóa ngoại và các phép xóa cha

Áp dụng cho toàn bộ FK đã liệt kê: bảng con (+ / − / + cột FK), bảng cha (− / + / + khóa được tham chiếu). Việc chuyển MaHD/MaCN/MaSP phải xét cả nhóm cũ và mới. GiaoDichKho phải có cặp (MaCN,MaSP) tồn tại trong TonKho theo QT-D2; triển khai FK ghép vào TonKho(MaCN,MaSP), nên xóa dòng tồn có nhật ký bị từ chối dù dấu xóa tại mệnh đề cân bằng trên các dòng tồn còn lại là −. Cấm xóa lịch sử vẫn áp dụng kể cả khi không có con tham chiếu.

### Tình huống đối chiếu GD1-13

- Hai dòng hóa đơn đều có đúng một mã hàng: xóa một dòng vẫn giữ D-R01 đúng; nếu không cập nhật tổng thì D-R02 có thể sai.
- Nhập +10, xuất −2, tồn 8: xóa xuất làm tổng nhật ký thành 10; xóa nhập làm tổng thành −2. Cả hai không khớp tồn 8: dấu xóa GiaoDichKho tại D-R03 là +.
- Chuyển nhật ký xuất −2 từ chi nhánh A sang B làm lệch cả hai nhóm tồn, nên sửa MaCN/MaSP phải nằm trong bảng ảnh hưởng.
- Hóa đơn bán cùng sản phẩm ở hai dòng số lượng 1 và 2: phải đối chiếu tổng xuất 3, không đối chiếu mỗi dòng với toàn nhật ký.
- Xóa dòng cuối hóa đơn phá số dòng tối thiểu; thêm dòng sau thanh toán phá bất biến dù điều kiện tối thiểu vẫn đúng.
- Xóa ghi nhận Loyalty có thể làm hóa đơn đã hoàn tất mất dấu chống lặp; ngừng bán sản phẩm không làm hóa đơn lịch sử vô hiệu.
- Xóa nhật ký không làm các giá trị còn lại vi phạm miền dấu số lượng, nhưng vẫn bị D-R03 và quy tắc bất biến chặn.

## Điểm ghép với A/B/C

- A xác nhận `ChiNhanh.MaCN`, `ChiNhanh.SucChua`, `NhanVien.MaNV` đúng kiểu/khóa; D dùng `MaCN` trên hóa đơn để xác định doanh thu và kho.
- B cung cấp nghiệp vụ cập nhật điểm/hạng theo MaHV, MaHD, TongTien, NgayLap và tạo GoiHoiVien trong cùng lần thanh toán. Lỗi phải hoàn tác toàn bộ dữ liệu thanh toán liên module; chữ ký thủ tục cụ thể thuộc GĐ2.
- C công bố thêm SoHieuCuaRa để D thống kê lượt tự đóng (TGRa có, cửa ra rỗng). D dùng công thức tại T và quy ước chất lượng dữ liệu ở QT-D5; giờ ra tự đóng là ước tính. C không kiểm tra SucChua trong luồng cổng.


GD1-09: D gọi B trong cùng giao dịch thanh toán; B kiểm tra đầu vào với HoaDon, ghi nhận duy nhất theo MaHD và cập nhật tổng/điểm/hạng nguyên tử. Hết 12 tháng chỉ loại tiền khỏi tổng xét hạng, giữ điểm. Hợp đồng tại [thiết kế điểm/hạng B](../b/diem-hang.md).

**GD1-11 — tạo gói từ dòng hóa đơn:** mỗi GoiHoiVien lưu (MaHD,STT,DonVi), FK dòng và UNIQUE bộ ba; dòng gói số lượng n tạo đúng n gói, dòng sản phẩm tạo 0. Xử lý lại cùng MaHD trả gói đã có; không tạo lại gói/cộng điểm/trừ kho. Hóa đơn/gói/điểm/kho phải cùng được ghi nhận đầy đủ hoặc cùng hoàn tác khi lỗi. Thông tin chủ và danh mục đọc qua vGoiHoiVien. Chi tiết tại [thiết kế B](../b/thiet-ke-logic.md).
