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
- [ERD Module C](erd-c.png)
- [Business Rules](business-rules.md)
- [Thiết kế logic](thiet-ke-logic.md)

## Phạm vi dữ liệu

- `CuaKiemSoat`: các cửa được đánh số trong phạm vi từng chi nhánh; khóa ghép `(MaCN, SoHieuCua)`.
- `CheckIn`: một phiên hiện diện của hội viên, từ lúc vào đến lúc ra.
- `LichHenPT`: lịch một giờ giữa hội viên và PT tại một chi nhánh.
- `DanhGiaHLV`: đánh giá gắn với một lịch PT đã hoàn thành.
- Module C chỉ tham chiếu dữ liệu chi nhánh/nhân sự của A và hội viên/gói của B; không sao chép thông tin danh mục của hai module này.

## Quyết định và giả định dùng trong thiết kế

- Đề yêu cầu mỗi check-in ghi nhận `Cửa kiểm soát/Thiết bị`. Thiết kế chọn mô hình cửa logic thay vì phần cứng: cửa được đánh số trong từng chi nhánh và log lưu cửa vào/ra để truy vết nguồn lượt quét. Đề không bắt buộc báo cáo theo cửa hoặc quản lý vòng đời thiết bị vật lý.
- Gói dùng được tại mọi chi nhánh theo quyết định #1. Quyền vào cổng và cách trừ buổi/vé dùng quyết định #2 và #8.
- Lịch PT dùng khung cố định một giờ; hủy trước hai giờ không mất buổi, hủy muộn hoặc không đến vẫn mất buổi; PT chỉ dạy tại chi nhánh đang công tác. Đây là giả định từ các quyết định #10–#12 đang **chưa chốt**.
- Một hội viên chỉ có tối đa một lượt check-in chưa đóng trên toàn hệ thống. Lượt quên quét ra được tự đóng theo giờ đóng cửa của chi nhánh; quyết định #15 vẫn cần nhóm xác nhận.
- `LichHenPT.DaTruBuoi` là cờ idempotency: một lịch chỉ làm giảm quyền lợi tối đa một lần, kể cả khi thao tác bị gửi lại.
- Hợp đồng hiện chưa công bố khóa của một gói đã mua (`MaGHV`) hoặc thủ tục trừ quyền lợi. C có thể kiểm tra bằng các cột đã chốt, nhưng trước giai đoạn 2 B và C phải công bố một khóa/thủ tục nguyên tử để C cập nhật đúng một `GoiHoiVien`, không chọn bản ghi mơ hồ.

## Điểm bàn giao cho module khác

- A đọc `LichHenPT(MaNV, MaCN, NgayGio, TrangThai)` để tính hoa hồng; chỉ trạng thái `Đã hoàn thành` được tính.
- D đọc `CheckIn(MaHV, MaCN, TGVao, TGRa)` để tính tỷ lệ lấp đầy.
- B cần xác nhận giao diện kiểm tra, giữ chỗ và trừ đúng một quyền lợi PT/Spa trước khi viết stored procedure ở giai đoạn 2.

## Ảnh hưởng liên module của `CuaKiemSoat`

- A vẫn chỉ công bố `ChiNhanh.MaCN`; C dùng cột này trong khóa ghép `(MaCN, SoHieuCua)`. A không phải thêm bảng/cột mới, nhưng không được đổi kiểu hoặc xóa mã chi nhánh đã được tham chiếu.
- B không đọc cửa kiểm soát và không bị ảnh hưởng.
- D tiếp tục dùng đúng hợp đồng `CheckIn(MaHV, MaCN, TGVao, TGRa)`; `SoHieuCuaVao`, `SoHieuCuaRa` là cột nội bộ của C nên D không phải sửa báo cáo lấp đầy.
