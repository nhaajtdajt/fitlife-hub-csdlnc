# Module C — Check-in và Lịch hẹn PT

| Mục | Nội dung |
|---|---|
| Người phụ trách | Module C |
| Phạm vi | Cửa kiểm soát, lượt vào/ra, lịch hẹn PT và đánh giá HLV |
| Vấn đề trọng tâm | Xác thực và ghi check-in dưới 50 ms trong giờ cao điểm; ngăn tuyệt đối hai hội viên đặt cùng một PT vào cùng khung giờ |
| Giai đoạn | Giai đoạn 1 — Phân tích, thiết kế mức quan niệm và logic |

## Tài liệu

- [Mô tả quy trình](quy-trinh.md)
- [Bảng phân tích tần suất](bang-tan-suat.md)
- [ERD Module C hiện hành](erd.md)
- [Business Rules](business-rules.md)
- [Thiết kế logic](thiet-ke-logic.md)

## Phạm vi dữ liệu

- `CuaKiemSoat`: các cửa được đánh số trong phạm vi từng chi nhánh; khóa ghép `(MaCN, SoHieuCua)`.
- `CheckIn`: một phiên hiện diện của hội viên, từ lúc vào đến lúc ra.
- `LichHenPT`: lịch một giờ gắn MaGHV của lần mua gói PT; hội viên suy ra từ gói. Không lưu MaHV lặp trên lịch.
- `DanhGiaHLV`: đánh giá gắn với một lịch PT đã hoàn thành.
- Module C chỉ tham chiếu dữ liệu chi nhánh/nhân sự của A và hội viên/gói của B; không sao chép thông tin danh mục của hai module này.

## Quyết định và giả định dùng trong thiết kế

- Đề yêu cầu mỗi check-in ghi nhận `Cửa kiểm soát/Thiết bị`. Thiết kế chọn mô hình cửa logic thay vì phần cứng: cửa được đánh số trong từng chi nhánh và log lưu cửa vào/ra để truy vết nguồn lượt quét. Đề không bắt buộc báo cáo theo cửa hoặc quản lý vòng đời thiết bị vật lý.
- Gói dùng được tại mọi chi nhánh theo quyết định #1. Quyền vào cổng và cách trừ buổi/vé dùng quyết định #2 và #8. Quyết định #9 chốt Gym/Yoga theo hạn gia hạn nối tiếp; B công bố thêm `NgayBatDau`, C phải xét cả hai biên ngày để loại gói chờ kích hoạt.
- Quyết định #2, #8, #11 cập nhật 08/11/2026: PT chỉ dùng đặt lịch; tự trừ tại giờ hẹn, độc lập vào cổng/tham dự. Chỉ hủy được khi còn ít nhất hai giờ; hủy muộn bị từ chối. Cổng dùng thẻ dịch vụ hoặc vé một lượt riêng. GD1-06/#12 đã chốt PT chỉ dạy tại chi nhánh công tác, xét chức vụ lịch sử và chặn đổi vị trí làm lịch chưa hủy không hợp lệ. QD10 đã chốt khung cố định một giờ, bắt đầu vào giờ tròn (phút/giây bằng 0).
- Một hội viên chỉ có tối đa một lượt check-in chưa đóng trên toàn hệ thống. Theo quy ước mô phỏng #15, lượt quên ra tự đóng tại giờ đóng cửa của ngày vào; đây là giờ ra ước tính. Chỉ báo cáo lấp đầy, không dùng SucChua để chặn cổng.
- `LichHenPT.DaTruBuoi` là cờ idempotency: một lịch chỉ làm giảm quyền lợi tối đa một lần bởi tác vụ đến giờ hẹn, kể cả khi tác vụ chạy lại. Cổng không đọc hoặc sửa cờ này.
- GD1-03 đã chốt MaGHV của từng lần mua. Lịch lưu MaGHV bắt buộc; đếm giữ chỗ từ lịch Đã đặt/chưa trừ trên từng gói, chọn hạn sớm nhất đủ ngày hẹn, kiểm tra lại quyền lợi và vị trí PT khi ghi nhận theo GD1-06. Chi tiết ở [thiết kế PT B–C](../b/goi-pt.md).
- `erd-c.png` là ảnh trước GD1-03, giữ để đối chiếu lịch sử; sơ đồ hiện hành dùng `erd.md` khi ghép báo cáo.

## Điểm bàn giao cho module khác

- A đọc `LichHenPT(MaNV, MaCN, NgayGio, TrangThai)` để tính hoa hồng; chỉ trạng thái `Đã hoàn thành` được tính.
- D đọc CheckIn(MaHV, MaCN, TGVao, TGRa, SoHieuCuaRa) để tính tỷ lệ tại T và thống kê lượt tự đóng.
- B/C đã chốt giao diện MaGHV và dữ liệu giữ/trừ PT. GD1-04 đã bỏ luồng kiểm tra Spa tại quầy: vé Spa dùng làm quyền vào chỉ trừ tại cổng, không cần thiết kế đối soát cổng–quầy. Thủ tục và kiểm thử SQL Server làm ở GĐ2.

## Ảnh hưởng liên module của `CuaKiemSoat`

- A vẫn chỉ công bố `ChiNhanh.MaCN`; C dùng cột này trong khóa ghép `(MaCN, SoHieuCua)`. A không phải thêm bảng/cột mới, nhưng không được đổi kiểu hoặc xóa mã chi nhánh đã được tham chiếu.
- B không đọc cửa kiểm soát và không bị ảnh hưởng.
- GD1-07 mở rộng hợp đồng cho D đọc thêm SoHieuCuaRa có sẵn để nhận diện lượt tự đóng; không thêm cột mới vào CheckIn. D đọc công thức và quy ước báo cáo ở QT-D5.

- GD1-05/#13 đã chốt: lịch sử công tác DATE tính cả hai biên, thay đổi trong ngày có hiệu lực ngày mai; A công bố ChucVu lịch sử. GD1-06 đã chốt: còn lịch chưa hủy không phù hợp thì từ chối đổi vị trí và trả danh sách lịch vướng; không tự hủy/chuyển lịch. C xét cả giai đoạn chờ khi nhận lịch mới.

**GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem [thiết kế B](../b/thiet-ke-logic.md).

Bản ghép hiện hành: [hồ sơ GĐ1 tổng](../README.md), [ERD tổng](../erd-tong.md), [lược đồ và ràng buộc tổng](../thiet-ke-tong.md).
