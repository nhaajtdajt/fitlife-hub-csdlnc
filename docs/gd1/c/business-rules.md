# Module C — Business Rules

Tài liệu này mô tả các quy tắc nghiệp vụ của Module C theo hai nhóm: quy tắc quan hệ/cardinality thể hiện trên ERD và các điều kiện nghiệp vụ bổ sung không thể hiện đầy đủ bằng đường nối ERD.

Ký hiệu: `1` là đúng một; `0..1` là không hoặc một; `0..N` là không hoặc nhiều; `1..N` là một hoặc nhiều.

## 1. Quy tắc quan hệ trên ERD

### BR-C01 — Chi nhánh và cửa kiểm soát

- Một `ChiNhanh` có từ một đến nhiều `CuaKiemSoat` (`1..N`).
- Mỗi `CuaKiemSoat` thuộc về đúng một `ChiNhanh` (`1`).
- Cửa được định danh duy nhất trong phạm vi chi nhánh bằng khóa ghép `(MaCN, SoHieuCua)`.

### BR-C02 — Chi nhánh và check-in

- Một `ChiNhanh` có thể phát sinh không hoặc nhiều `CheckIn` (`0..N`).
- Mỗi `CheckIn` phát sinh tại đúng một `ChiNhanh` (`1`).

### BR-C03 — Cửa vào và check-in

- Một `CuaKiemSoat` có thể được dùng làm cửa vào của không hoặc nhiều `CheckIn` (`0..N`).
- Mỗi `CheckIn` phải được ghi nhận tại đúng một cửa vào (`1`).
- Cửa vào được tham chiếu bằng khóa ngoại ghép `(MaCN, SoHieuCuaVao)`.

### BR-C04 — Cửa ra và check-in

- Một `CuaKiemSoat` có thể được dùng làm cửa ra của không hoặc nhiều `CheckIn` (`0..N`).
- Mỗi `CheckIn` có không hoặc một cửa ra (`0..1`).
- Cửa ra được tham chiếu bằng khóa ngoại ghép `(MaCN, SoHieuCuaRa)`; `SoHieuCuaRa` được phép rỗng khi hội viên chưa ra hoặc hệ thống tự ghi `TGRa`.

### BR-C05 — Hội viên và check-in

- Một `HoiVien` có thể thực hiện không hoặc nhiều `CheckIn` (`0..N`).
- Mỗi `CheckIn` thuộc về đúng một `HoiVien` (`1`).

### BR-C06 — Hội viên và lịch hẹn PT

- Một `HoiVien` có thể đặt không hoặc nhiều `LichHenPT` (`0..N`).
- Mỗi `LichHenPT` thuộc về đúng một `HoiVien` (`1`).

### BR-C07 — Chi nhánh và lịch hẹn PT

- Một `ChiNhanh` có thể là nơi diễn ra không hoặc nhiều `LichHenPT` (`0..N`).
- Mỗi `LichHenPT` diễn ra tại đúng một `ChiNhanh` (`1`).

### BR-C08 — Nhân viên và lịch hẹn PT

- Một `NhanVien` có thể phụ trách không hoặc nhiều `LichHenPT` (`0..N`).
- Mỗi `LichHenPT` do đúng một `NhanVien` phụ trách (`1`).
- Nhân viên được chọn phải có chức vụ PT tại thời điểm của lịch hẹn.

### BR-C09 — Lịch hẹn PT và đánh giá HLV

- Một `LichHenPT` có không hoặc một `DanhGiaHLV` (`0..1`).
- Mỗi `DanhGiaHLV` thuộc về đúng một `LichHenPT` (`1`).
- `DanhGiaHLV.MaLichHen` phải `UNIQUE` để một lịch không có hai đánh giá.

## 2. Quy tắc nghiệp vụ bổ sung

### BR-C10 — Phiên check-in đang mở

- Một hội viên có tối đa một `CheckIn` có `TGRa` rỗng trên toàn hệ thống.
- `TGRa IS NULL` nghĩa là hội viên đang được xem là còn trong phòng tập.
- Nếu `TGRa` có giá trị thì `TGRa` phải lớn hơn `TGVao`.
- `SoHieuCuaRa` rỗng không đồng nghĩa hội viên còn trong phòng nếu `TGRa` đã có giá trị.

### BR-C11 — Cửa hợp lệ

- `SoHieuCua` là số nguyên dương và không được trùng trong cùng một chi nhánh.
- Cửa nhận lượt quét mới phải có `DangHoatDong = true`.
- Cửa vào phải có `ChucNang` là `Vào` hoặc `Hai chiều`.
- Cửa ra phải có `ChucNang` là `Ra` hoặc `Hai chiều`.
- Không xóa vật lý cửa đã được check-in tham chiếu; chỉ chuyển trạng thái ngừng hoạt động.

### BR-C12 — Điều kiện cho phép check-in

- Chỉ tạo `CheckIn` khi hội viên có quyền vào hợp lệ tại `TGVao`: Gym/Yoga còn hạn, vé Spa còn lượt, hoặc gói PT còn hạn/còn buổi kèm lịch PT hợp lệ trong ngày.
- Đây là điều kiện kiểm tra dữ liệu của B tại thời điểm giao dịch, không phải quan hệ khóa ngoại được lưu trong `CheckIn`.
- Kiểm tra quyền lợi, trừ lượt nếu cần và tạo `CheckIn` phải cùng thành công hoặc cùng thất bại.

### BR-C13 — Tự đóng lượt quên quét ra

- Theo đề xuất #15 đang chờ nhóm chốt, các lượt còn mở được ghi `TGRa` tại giờ đóng cửa của chi nhánh.
- Lượt tự đóng không có thao tác quét ra nên `SoHieuCuaRa` giữ rỗng.
- Tác vụ phải chạy lặp an toàn và không cập nhật lại lượt đã có `TGRa`.

### BR-C14 — Không trùng lịch PT

- Không tồn tại hai `LichHenPT` có cùng `MaNV`, `NgayGio` khi cả hai đều chưa hủy.
- Khi hai giao dịch đồng thời đặt cùng PT/khung giờ, tối đa một giao dịch được thành công.
- Mỗi khung PT kéo dài một giờ theo đề xuất #10 đang chờ nhóm chốt.

### BR-C15 — Điều kiện đặt lịch PT

- Tại ngày hẹn, hội viên phải có gói PT còn hạn/còn buổi và gói Gym/Yoga còn hạn.
- Số lịch chưa hủy đang giữ quyền lợi không được vượt số buổi PT khả dụng.
- Các điều kiện này đọc dữ liệu gói của B nhưng không tạo quan hệ ERD nếu `LichHenPT` không lưu `MaGHV`.

### BR-C16 — PT thuộc đúng chi nhánh

- Tại `NgayGio`, nhân viên phụ trách phải có đúng một giai đoạn công tác hợp lệ tại `MaCN` của lịch.
- Điều kiện được kiểm tra qua `LichSuCongTac` của A; `LichHenPT` không lưu khóa ngoại đến một bản ghi lịch sử công tác nên quan hệ này không vẽ trên ERD C.

### BR-C17 — Chuyển trạng thái lịch

- `TrangThai` chỉ thuộc `Đã đặt`, `Đã hoàn thành`, `Hủy`.
- Chỉ cho phép `Đã đặt → Đã hoàn thành` hoặc `Đã đặt → Hủy`.
- Lịch đã hoàn thành hoặc đã hủy không được chuyển trở lại `Đã đặt`.

### BR-C18 — Trừ buổi PT đúng một lần

- `DaTruBuoi` chỉ được chuyển từ 0 sang 1.
- Một lịch chỉ làm giảm quyền lợi PT tối đa một lần, kể cả khi yêu cầu bị gửi lại.
- Lịch hoàn thành, hủy muộn hoặc không đến phải có `DaTruBuoi = 1`; lịch hủy đúng hạn có `DaTruBuoi = 0`, theo đề xuất #11 đang chờ nhóm chốt.

### BR-C19 — Đánh giá HLV

- Chỉ `LichHenPT` có trạng thái `Đã hoàn thành` mới được đánh giá.
- Người gửi đánh giá phải là hội viên của lịch.
- `Diem` là số nguyên từ 1 đến 5.
- `ThoiDiemDanhGia` không được trước thời điểm buổi tập hoàn thành.

### BR-C20 — Bảo toàn dữ liệu lịch sử

- Không xóa vật lý `CheckIn`, lịch đã hoàn thành/hủy hoặc đánh giá đã dùng cho báo cáo và tính hoa hồng.
- Nếu phải điều chỉnh dữ liệu lịch sử, thao tác phải được phân quyền và có khả năng truy vết ở giai đoạn vật lý.
