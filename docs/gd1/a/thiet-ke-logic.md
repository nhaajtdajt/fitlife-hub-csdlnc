# Thiết kế logic Module A — GD1-10

Phạm vi: chi nhánh, tiện ích và thời gian cung cấp, nhân viên, lịch sử công tác, kỳ hoa hồng đã chốt. Kiểu/độ dài SQL và chỉ mục thuộc GĐ2. Quy tắc đã chốt #6, #12–15; quy trình đổi vị trí theo GD1-05/06 giữ nguyên.

## Lược đồ, khóa và phụ thuộc hàm

Trong bảng, K → phần còn lại nghĩa là khóa xác định tất cả thuộc tính ngoài khóa. Không mặc định tên/SĐT là duy nhất.

| Quan hệ và thuộc tính | PK; khóa ứng viên khác | PTH và dạng chuẩn |
|---|---|---|
| ChiNhanh(MaCN, TenCN, DiaChi, SDT, GioMoCua, GioDongCua, SucChua, HoaHongMoiBuoi, TrangThai) | MaCN; không có khóa khác được giả định | MaCN → phần còn lại; BCNF |
| TienIch(MaTI, TenTI, Loai, MoTa, TrangThai) | MaTI | MaTI → phần còn lại; BCNF |
| CungCapTienIch(MaCN, MaTI, NgayBatDau, NgayKetThuc) | (MaCN, MaTI, NgayBatDau) | Khóa → NgayKetThuc; BCNF |
| NhanVien(MaNV, HoTen, SDT, ChucVu, LuongCoBan, TrangThaiLamViec) | MaNV | MaNV → phần còn lại; BCNF theo PTH nội quan hệ này |
| LichSuCongTac(MaLS, MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc, LyDo) | MaLS; AK (MaNV, NgayBatDau) | Mỗi khóa → phần còn lại; BCNF |
| KyHoaHong(MaCN, Thang, MucHoaHongChot, ThoiDiemChot, MaNVChot) | (MaCN, Thang) | Khóa → phần còn lại; BCNF |
| ChiTietHoaHong(MaLichHen, MaCN, Thang) | MaLichHen | MaLichHen → MaCN, Thang; BCNF |

Thang là DATE ngày đầu tháng, kỳ từ đầu tháng đến trước đầu tháng kế tiếp, Asia/Saigon. KyHoaHong chỉ chứa kỳ đã chốt, không thêm cờ trạng thái hoặc tổng tiền/số buổi lưu lặp. ChiTietHoaHong là danh sách buổi được chốt; mỗi buổi được tính tối đa một lần. Thành tiền của mỗi buổi chính là MucHoaHongChot của kỳ; số buổi và tổng theo nhân viên được tính bằng COUNT/SUM, không lưu thuộc tính dẫn xuất tạo PTH ngoài khóa.

Khóa ngoại: CungCapTienIch.MaCN → ChiNhanh, MaTI → TienIch; LichSuCongTac.MaNV → NhanVien, MaCN → ChiNhanh; KyHoaHong.MaCN → ChiNhanh, MaNVChot → NhanVien; ChiTietHoaHong.MaLichHen → C.LichHenPT, (MaCN, Thang) → KyHoaHong. Mọi FK bắt buộc, không cascade xóa lịch sử. Quan hệ C dùng: LichHenPT(MaLichHen, MaNV, MaCN, NgayGio, TrangThai).

## Ánh xạ ERD và ý nghĩa thuộc tính

- ChiNhanh: mã ổn định; tên/địa chỉ/SĐT liên hệ; giờ mở/đóng TIME trong cùng ngày theo #15; SucChua nguyên dương chỉ phục vụ báo cáo; HoaHongMoiBuoi số đồng nguyên không âm là mức đang cấu hình; TrangThai hoạt động/ngừng hoạt động.
- TienIch: danh mục tiện ích/dịch vụ cơ sở vật chất, không đồng nhất với GoiDichVu là danh mục bán. TenTI bắt buộc, Loai là nhóm tiện ích, MoTa tùy chọn, TrangThai đang dùng/ngừng dùng. CungCapTienIch ánh xạ quan hệ n–n có thời gian của ERD, cho phép ngừng rồi cung cấp lại. Ngày DATE bao gồm hai biên, kết thúc rỗng nếu chưa xác định; cặp chi nhánh–tiện ích không có hai khoảng giao nhau.
- NhanVien: HoTen bắt buộc, SDT liên hệ; LuongCoBan không âm là lương hiện tại, không tuyên bố phục vụ tính lương quá khứ; TrangThaiLamViec đang làm/nghỉ việc. ChucVu là PT/Lễ tân/Quản lý chi nhánh/Nhân viên bán hàng. Phân cấp toàn phần, loại trừ của ERD ánh xạ bằng cột ChucVu, không cần bốn bảng con chỉ chứa mã.
- LichSuCongTac: MaLS ổn định; ChucVu và MaCN tại giai đoạn; LyDo mô tả tiếp nhận/điều chuyển/đổi chức vụ, có thể rỗng khi tiếp nhận. Ngày DATE bao gồm hai biên. Một ngày hợp lệ; mới bắt đầu ngày sau ngày kết thúc cũ. Chức vụ hiện tại là giá trị đồng bộ liên bảng, không phải PTH nội bảng NhanVien bổ sung. Chi nhánh hiện tại suy ra từ lịch sử, không lưu thêm MaCN trên nhân viên.
- KyHoaHong: MucHoaHongChot là bản sao mức của chi nhánh tại lúc xác nhận chốt; ThoiDiemChot ghi thời điểm xác nhận, MaNVChot ghi người có quyền chốt. Đây là chính sách mức cho kỳ chưa chốt của thiết kế hiện tại, chưa có lịch sử thay đổi mức theo từng ngày dạy.
- ChiTietHoaHong: chỉ lưu mã buổi và khóa kỳ; nhân viên, ngày giờ truy qua LichHenPT. Sau chốt phải bảo vệ các trường nguồn đó để không đổi kết quả truy xuất kỳ cũ.

## Chốt kỳ và bảo toàn kết quả

1. Chỉ chốt tháng đã kết thúc. Xem trước là truy vấn, chưa ghi kỳ. Kiểm tra kỳ (MaCN, Thang) chưa được chốt; mức hoa hồng phải đúng tại thời điểm xác nhận chốt.
2. Đọc lại toàn bộ lịch thuộc chi nhánh/tháng. Nếu còn Đã đặt chưa phân loại kết quả, hoặc lịch hoàn thành không khớp đúng một giai đoạn PT/chi nhánh bao phủ buổi, từ chối chốt và liệt kê dòng cần xử lý. Lịch Hủy không tính hoa hồng; lịch hoàn thành hợp lệ mới được đưa vào chi tiết.
3. Tạo KyHoaHong với mức đã đọc và toàn bộ ChiTietHoaHong trong cùng giao dịch. Lỗi hoàn tác tất cả; tháng không có buổi hợp lệ vẫn được chốt với không dòng chi tiết. Chạy lại thấy kỳ đã có thì trả kỳ cũ, không tính/lưu thêm.
4. Sau chốt, xem/xuất từ kỳ và danh sách chi tiết, không đọc HoaHongMoiBuoi hiện tại để tính lại. Ví dụ 10 buổi × mức chốt 50.000đ = 500.000đ; sửa mức hiện tại thành 60.000đ vẫn trả 500.000đ cho kỳ cũ.
5. Không mở lại/sửa/xóa kỳ hoặc chi tiết trong phạm vi hiện tại. Chặn sửa MaNV/MaCN/NgayGio/TrangThai hoặc xóa buổi của kỳ đã chốt, và chặn thay đổi lịch sử công tác làm mất căn cứ của buổi đó. Không thêm/đưa buổi mới thành hoàn thành vào kỳ đã chốt. Sửa mức chi nhánh chỉ ảnh hưởng lần xem trước/chốt kỳ chưa chốt tiếp theo.
6. Chốt kỳ và cập nhật lịch/kết quả của C phải nhất quán: không để buổi mới hoặc sửa nguồn lọt vào kỳ đã chốt. Mỗi kỳ chỉ chốt một lần, mỗi buổi chỉ thuộc một kỳ; yêu cầu lặp trả kết quả cũ. Luồng hiện tại không tự chuyển lịch.

## Ràng buộc: bối cảnh, nội dung, tầm ảnh hưởng

Dấu + là thao tác có thể vi phạm, − là không trực tiếp vi phạm mệnh đề; bảng được nêu theo Thêm/Xóa/Sửa. Cấm xóa lịch sử là quy tắc riêng, không thay dấu của ràng buộc miền.

| ID | Bối cảnh/nội dung | Tầm ảnh hưởng |
|---|---|---|
| A-R01 | ChiNhanh: SucChua nguyên > 0, mức hoa hồng >= 0, giờ mở < giờ đóng; trạng thái hợp lệ | ChiNhanh (+/−/+ các cột miền) |
| A-R02 | TienIch/NhanVien: tên bắt buộc, trạng thái/chức vụ đúng tập, lương >= 0 | TienIch (+/−/+ TenTI, TrangThai); NhanVien (+/−/+ HoTen, ChucVu, LuongCoBan, TrangThaiLamViec) |
| A-R03 | FK tồn tại; không có dữ liệu mồ côi | Bảng con (+/−/+ FK); bảng cha (−/+/+ khóa được tham chiếu) |
| A-R04 | CungCapTienIch: kết thúc >= bắt đầu, các khoảng cùng (MaCN,MaTI) không chồng, ngày kết thúc rỗng là vô hạn | CungCapTienIch (+/−/+ MaCN,MaTI,NgayBatDau,NgayKetThuc) |
| A-R05 | LichSuCongTac: ngày hợp lệ, không chồng theo nhân viên; tối đa một thay đổi đang chờ | LichSuCongTac (+/−/+ MaNV,NgayBatDau,NgayKetThuc); thời gian thay đổi tính hiện hành |
| A-R06 | Nhân viên đang làm phải có đúng một giai đoạn hôm nay; NhanVien.ChucVu khớp giai đoạn đó sau đồng bộ | NhanVien (+/−/+ ChucVu,TrangThaiLamViec); LichSuCongTac (+/+/+ MaNV,ChucVu,NgayBatDau,NgayKetThuc); sang ngày mới kích hoạt đồng bộ |
| A-R07 | Đổi vị trí từ ngày mai, không ghi đè lịch sử; từ chối nếu còn lịch PT chưa hủy không phù hợp theo GD1-06 | LichSuCongTac (+/+/+ các cột vị trí/ngày); LichHenPT (+/−/+ MaNV,MaCN,NgayGio,TrangThai) |
| A-R08 | Kỳ có Thang là đầu tháng đã kết thúc, mức không âm, người chốt hợp lệ; chỉ chốt một lần | KyHoaHong (+/−/+ Thang,MucHoaHongChot,MaNVChot,ThoiDiemChot); PK chống trùng |
| A-R09 | Chi tiết đúng kỳ/chi nhánh/ngày, buổi đã hoàn thành, lịch sử có PT đúng nơi dạy; mỗi buổi một lần | ChiTietHoaHong (+/−/+); LichHenPT (−/+/+ MaNV,MaCN,NgayGio,TrangThai); LichSuCongTac (+/+/+ các cột đối chiếu); KyHoaHong (−/+/+ khóa) |
| A-R10 | Kỳ đã chốt chứa đủ buổi hợp lệ, không bỏ sót; không nhận nguồn mới vào kỳ đó | ChiTietHoaHong (+/+/+); LichHenPT (+/+/+ các cột kỳ/kết quả); LichSuCongTac (+/+/+ các cột đối chiếu) |
| A-R11 | Kỳ/chi tiết đã chốt bất biến; lịch sử đã tham chiếu không xóa; nguồn buổi đã chốt được bảo vệ | KyHoaHong (−/+/+); ChiTietHoaHong (+/+/+); LichHenPT (−/+/+ các cột nguồn); LichSuCongTac (−/+/+ các cột giai đoạn đã dùng) |

## Kết luận

Bảy quan hệ đạt BCNF theo các PTH đã công bố: định thức đều là khóa; không lưu số buổi × mức = thành tiền trong cùng quan hệ để rồi bỏ sót PTH dẫn xuất. Đồng bộ chức vụ hiện tại/lịch sử, không chồng khoảng và tính đầy đủ/bất biến kỳ là ràng buộc liên bảng/thời gian, cần thực thi bằng giao dịch. Kết luận toàn dự án tại [thiết kế tổng GD1-17](../thiet-ke-tong.md); toàn CSDL chỉ bảo đảm 2NF vì HoiVien lưu hạng theo tổng chi tiêu.
