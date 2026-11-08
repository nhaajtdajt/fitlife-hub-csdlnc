# Thiết kế logic B — GD1-11

Tích hợp PT (GD1-03), quyền vào cổng (GD1-04), điểm/hạng (GD1-09) và liên kết dòng hóa đơn đã chốt. Các tài liệu goi-pt.md/diem-hang.md mô tả quy tắc chi tiết; tài liệu này chốt cấu trúc cơ sở và ánh xạ giao diện đọc.

## Lược đồ, khóa, phụ thuộc hàm và dạng chuẩn

| Quan hệ | PK / AK | PTH và dạng chuẩn |
|---|---|---|
| HoiVien(MaHV, HoTen, SDT, Email, CCCD, GioiTinh, NgaySinh, NgayDangKy, MaCNDangKy, TongChiTieu12T, DiemTichLuy, Hang) | PK MaHV; AK CCCD, SDT (bắt buộc, duy nhất theo QT-B1) | Mỗi khóa → phần còn lại; TongChiTieu12T → Hang theo ngưỡng cố định. Đạt 2NF, không đạt 3NF/BCNF; lưu hạng dư thừa có chủ đích để đọc nhanh theo GD1-09 |
| GoiDichVu(MaGoi, TenGoi, LoaiGoi, CachTinh, ThoiHanThang, SoBuoiBanDau, Gia, DangKinhDoanh) | PK MaGoi | MaGoi → phần còn lại; BCNF theo PTH đã nêu; kiểm tra tổ hợp loại/cách tính bằng ràng buộc miền |
| GoiHoiVien(MaGHV, MaHD, STT, DonVi, NgayBatDau, NgayHetHan, SoBuoiConLai) | PK MaGHV; AK (MaHD, STT, DonVi) | Mỗi khóa → phần còn lại; BCNF theo PTH đã nêu |
| GhiNhanLoyalty(MaHD, DiemCong, ThoiDiemGhiNhan, DaLoaiKhoi12T) | PK MaHD | MaHD → phần còn lại; BCNF; DiemCong tính từ tiền ở bảng khác, không tạo PTH ngoài khóa trong quan hệ này |

FK: HoiVien.MaCNDangKy → A.ChiNhanh.MaCN; GoiHoiVien.(MaHD,STT) → D.ChiTietHoaDon.(MaHD,STT); GhiNhanLoyalty.MaHD → D.HoaDon.MaHD. C.LichHenPT.MaGHV tiếp tục FK tới GoiHoiVien.MaGHV. Không cascade xóa lịch sử.

Ngoài module: HoaDon(MaHD,MaHV,NgayLap,TongTien), ChiTietHoaDon(MaHD,STT,LoaiDong,MaGoi,MaSP,SoLuong,DonGia), ChiNhanh(MaCN), LichHenPT(MaLichHen,MaGHV,NgayGio,TrangThai,DaTruBuoi), CheckIn(MaHV,MaCN,TGVao,TGRa).

## Ánh xạ dữ liệu đọc và chuẩn hóa

Không lưu lặp MaHV/MaGoi trên GoiHoiVien cơ sở: MaHD đã xác định MaHV, (MaHD,STT) đã xác định MaGoi; thêm hai cột đó sẽ có phụ thuộc vào một phần khóa ứng viên, không còn 2NF. Quan hệ dẫn xuất vGoiHoiVien nối GoiHoiVien → ChiTietHoaDon → HoaDon, công bố MaGHV, MaHV, MaGoi, NgayBatDau, NgayHetHan, SoBuoiConLai và khóa dòng/đơn vị. Đây là giao diện thay thế các danh sách cột GoiHoiVien có MaHV/MaGoi trong tài liệu PT/C trước GD1-11. Trừ số dư vẫn tác động đúng MaGHV trên bảng cơ sở GoiHoiVien. vGoiHoiVien ở GĐ1 là mô tả phép nối logic; lựa chọn SQL View hay truy vấn thuộc GĐ2.

HoiVien được giữ đúng phương án lưu tổng/điểm/hạng đã chốt. Phương án chuẩn hóa bỏ Hang khỏi bảng cơ sở và tính từ TongChiTieu12T qua quan hệ dẫn xuất sẽ loại PTH không khóa; ở đây chủ động giữ Hang làm bộ nhớ tổng hợp, bắt buộc đồng bộ theo diem-hang.md. Không kết luận toàn B đạt BCNF. Không tạo bảng cho từng số tiền để chỉ che phụ thuộc TongChiTieu12T → Hang.

## Thuộc tính và miền

- HoiVien: mã ổn định; họ tên bắt buộc; SDT/CCCD bắt buộc duy nhất, CCCD 12 chữ số; email/giới tính theo dữ liệu hồ sơ, ngày sinh không tương lai; ngày đăng ký và chi nhánh đăng ký lưu lúc tiếp nhận, chi nhánh chỉ dùng phân nhóm báo cáo, không hạn chế quyền vào. Tổng chi tiêu/điểm không âm; hạng Silver/Gold/Platinum. Thay đổi thông tin liên hệ không xóa lịch sử.
- GoiDichVu: TenGoi bắt buộc; LoaiGoi = Gym/Yoga/PT/Spa; CachTinh = TheoHan/TheoBuoi. Gym/Yoga TheoHan có 1/3/6/12 tháng và số buổi rỗng; Gym/Yoga TheoBuoi và Spa có 1 lượt, thời hạn rỗng; PT TheoBuoi có số buổi nguyên dương và hạn 3/6/12 tháng. Gia là số đồng nguyên dương; DangKinhDoanh bật/tắt bán. Thay đổi giá không thay đơn giá hóa đơn cũ.
- Gói danh mục đã bán không sửa LoaiGoi/CachTinh/ThoiHanThang/SoBuoiBanDau: muốn đổi quyền lợi tạo mã danh mục mới. Được đổi giá/ngừng bán. Quyền lợi đã mua vẫn giữ.
- GoiHoiVien: MaGHV duy nhất một đơn vị gói; STT là số dòng hóa đơn, DonVi nguyên 1..SoLuong của dòng; không phải mã danh mục. Ngày DATE tính cả hai biên. Gym/Yoga nối tiếp, PT bắt đầu ngày mua, vé dùng từ ngày mua; công thức hạn và giữ/trừ buổi tại goi-pt.md/quy-trinh.md. TheoHan có SoBuoiConLai rỗng; PT/lượt có số nguyên không âm, không vượt số buổi ban đầu; vé có NgayHetHan rỗng.
- GhiNhanLoyalty: mã hóa đơn là dấu ghi nhận duy nhất, DiemCong nguyên không âm, thời điểm ghi nhận, cờ loại khỏi tổng xét hạng. Dữ liệu chủ hội viên/tiền/ngày lấy từ hóa đơn bất biến; hết 12 tháng chỉ đổi cờ và tổng, không xóa điểm.

## Tạo gói từ hóa đơn, nguyên tử và chống lặp

1. D cấp MaHD ổn định cho lần thanh toán và STT cho từng dòng. Xử lý lại cùng giao dịch phải dùng lại MaHD; yêu cầu có MaHD khác là giao dịch mua khác, không tự gộp theo nội dung giống nhau.
2. Trong một giao dịch, kiểm tra lại chủ hóa đơn, LoaiDong = GoiDichVu, MaGoi tồn tại/được bán và SoLuong nguyên dương. Một dòng sản phẩm tuyệt đối không tạo GoiHoiVien.
3. Dòng gói có SoLuong = n tạo đúng n bản ghi, DonVi lần lượt 1..n, mỗi bản ghi có MaGHV riêng. Liên kết (MaHD,STT) không đổi. Cùng dòng mua 2 PT tạo 2 gói riêng; cùng dòng mua 2 Gym TheoHan tạo 2 kỳ nối tiếp theo thứ tự STT/DonVi. Các lần mua đồng thời không được tạo kỳ chồng nhau; xác nhận lại ngày nếu dự kiến thay đổi.
4. AK (MaHD,STT,DonVi) ngăn trùng. Đã hoàn tất MaHD thì trả lại hóa đơn và danh sách gói hiện có, không tạo mới, không cộng điểm hoặc trừ kho lần nữa. Nếu thấy số lượng/liên kết đã ghi không khớp thì báo dữ liệu bất thường, không âm thầm bỏ qua hay tự tạo phần thiếu của hóa đơn đã chốt.
5. Hóa đơn, tất cả gói, ghi nhận Loyalty/tổng/hạng và tồn kho cùng commit hoặc rollback. GhiNhanLoyalty vẫn một dòng mỗi hóa đơn, không một dòng mỗi gói. Cổng/đặt lịch chỉ thấy gói sau commit. Không ghi một hóa đơn đã thanh toán còn thiếu đơn vị gói.
6. Dòng và đầu hóa đơn đã thanh toán không sửa/xóa; gói không đổi liên kết/ngày/chủ/loại. Việc thanh toán và cập nhật quyền lợi phải nhất quán, không ghi nhận một phần khi lỗi.

## Ràng buộc và tầm ảnh hưởng

Dấu +/− theo Thêm/Xóa/Sửa; cấm xóa lịch sử được ghi riêng.

| ID | Bối cảnh/nội dung | Tầm ảnh hưởng |
|---|---|---|
| B-R01 | Hồ sơ bắt buộc, SDT/CCCD duy nhất, ngày hợp lệ | HoiVien (+/−/+ cột hồ sơ); PK/AK kiểm tra trùng |
| B-R02 | Danh mục đúng tổ hợp loại/cách tính/hạn/số buổi, giá dương | GoiDichVu (+/−/+ các cột miền) |
| B-R03 | FK hợp lệ | Bảng con (+/−/+ FK); bảng cha (−/+/+ khóa) |
| B-R04 | Mỗi gói tham chiếu dòng loại gói; DonVi trong 1..SoLuong; mỗi đơn vị chỉ một gói | GoiHoiVien (+/−/+ MaHD,STT,DonVi); ChiTietHoaDon (−/+/+ khóa,LoaiDong,MaGoi,SoLuong) |
| B-R05 | Hóa đơn đã thanh toán có đủ n gói cho mỗi dòng gói, không gói cho dòng sản phẩm | GoiHoiVien (+/+/+ liên kết); ChiTietHoaDon (+/+/+ loại,số lượng,mã gói); HoaDon (+/+/+ khóa/chủ) |
| B-R06 | Gym/Yoga TheoHan cùng hội viên/loại không chồng; ngày/buổi đúng loại, số dư không âm | GoiHoiVien (+/−/+ ngày,số dư,liên kết); HoaDon/ChiTietHoaDon/GoiDichVu (−/+/+ các cột xác định chủ/loại/quyền lợi) |
| B-R07 | Giữ PT không vượt số dư; đến giờ trừ đúng MaGHV một lần, độc lập cổng | LichHenPT (+/−/+ gói,trạng thái,cờ,giờ); GoiHoiVien (−/+/+ số dư/ngày/liên kết); chi tiết tại GD1-03 |
| B-R08 | Tiền/điểm/hạng và ghi nhận khớp hóa đơn tại lúc đồng bộ, giữ điểm khi hết cửa sổ | HoiVien (+/−/+ tổng,điểm,hạng); GhiNhanLoyalty (+/+/+); HoaDon (−/+/+ MaHV,TongTien,NgayLap); thời gian kích hoạt đồng bộ |
| B-R09 | Gói/liên kết/quyền lợi danh mục đã bán bất biến; lịch sử mua/điểm không xóa | GoiHoiVien (−/+/+ liên kết/ngày); GoiDichVu (−/+/+ cột quyền lợi); GhiNhanLoyalty (−/+/+ dữ liệu gốc); số dư/cờ thay đổi chỉ theo giao dịch hợp lệ |

## Kết luận và giới hạn

Ba quan hệ GoiDichVu/GoiHoiVien/GhiNhanLoyalty đạt BCNF theo PTH đã nêu; HoiVien đạt 2NF và lưu hạng dư thừa có chủ đích. View không làm phát sinh dữ liệu chủ/gói lặp ở bảng cơ sở. Các ngưỡng hạng 20/50 triệu vẫn cần cô xác nhận; chưa triển khai bảo lưu/đổi điểm. Những giới hạn này được giữ trong sổ theo dõi, không coi là đã chốt thêm nghiệp vụ. Kết luận toàn CSDL tại [thiết kế tổng GD1-17](../thiet-ke-tong.md): mức chung bảo đảm 2NF, không kết luận toàn CSDL BCNF.
