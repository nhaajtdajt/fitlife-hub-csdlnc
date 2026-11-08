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

### BR-C06 — Hội viên, gói đã mua và lịch hẹn PT

- Một HoiVien có 0..N GoiHoiVien; mỗi gói đã mua thuộc đúng một hội viên.
- Một GoiHoiVien có 0..N LichHenPT; mỗi lịch tham chiếu đúng một gói qua MaGHV bắt buộc. Chỉ gói loại PT được gắn lịch.
- Hội viên của lịch suy ra qua vGoiHoiVien.MaHV; không lưu thêm MaHV trong LichHenPT. Không đổi chủ gói hoặc MaGHV của lịch sau khi đã tạo.

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

- Chỉ tạo `CheckIn` khi hội viên có quyền vào hợp lệ tại `TGVao`: thẻ dịch vụ theo hạn còn hiệu lực hoặc vé vào một lượt còn lượt. Gói PT không cấp quyền vào cổng; cổng không đọc lịch hoặc trừ PT.
- Thẻ theo hạn phải thỏa `NgayBatDau <= Ngay(TGVao) <= NgayHetHan`; gói Gym/Yoga gia hạn nối tiếp chưa bắt đầu không cấp quyền vào. Vé một lượt đã mua xét lượt còn lại, không so với hạn rỗng.
- SucChua chỉ là mẫu số báo cáo, không tham gia điều kiện vào cổng; tỷ lệ vượt 100% không làm từ chối một quyền vào hợp lệ.
- Đây là điều kiện kiểm tra dữ liệu của B tại thời điểm giao dịch, không phải quan hệ khóa ngoại được lưu trong `CheckIn`.
- Kiểm tra quyền vào, trừ vé nếu cần và tạo `CheckIn` phải cùng thành công hoặc cùng thất bại. Vé một lượt đã dùng không được tái sử dụng sau khi ra, dù vẫn có lịch PT; muốn vào lại cần quyền vào mới. Quét lặp trong phiên đang mở không trừ thêm.

### BR-C13 — Tự đóng lượt quên quét ra

- Theo quy ước mô phỏng #15, chi nhánh hoạt động trong cùng ngày, giờ mở < giờ đóng và cấu hình cố định trong kỳ dữ liệu. Lượt quên ra được ghi TGRa ước tính tại giờ đóng cửa của đúng ngày vào; không lấy thời điểm chạy tác vụ nếu chạy bù.
- Lượt tự đóng không có thao tác quét ra nên `SoHieuCuaRa` giữ rỗng.
- Tác vụ phải chạy lặp an toàn, đọc lại trạng thái trong giao dịch, không ghi đè lượt đã có TGRa hoặc phiên của ngày mới. Lượt ngoài giờ cấu hình cần đối soát. Báo cáo còn lượt quá hạn chưa đóng là tạm tính.

### BR-C14 — Không trùng lịch PT

- Không tồn tại hai `LichHenPT` có cùng `MaNV`, `NgayGio` khi cả hai đều chưa hủy.
- Khi hai giao dịch đồng thời đặt cùng PT/khung giờ, tối đa một giao dịch được thành công.
- QD10 đã chốt: mỗi khung kéo dài một giờ, bắt đầu vào giờ tròn theo Asia/Saigon; phút/giây bằng 0. Hai khung liền nhau không chồng lấn; không chấp nhận giờ bắt đầu như 17h30.

### BR-C15 — Điều kiện đặt lịch PT

- Tại ngày hẹn, hội viên phải có gói PT thỏa `NgayBatDau <= NgayHen <= NgayHetHan` và buổi chưa bị giữ cho lịch khác; không kiểm tra thẻ vào cổng tại bước đặt lịch.
- Số lịch chưa hủy, chưa trừ đang giữ quyền lợi trên từng gói không được vượt số buổi còn lại của gói đó.
- MaGHV là FK bắt buộc trong LichHenPT. Gói phải thuộc hội viên đặt; chọn gói đủ hạn tại ngày hẹn và còn khả dụng, theo NgayHetHan rồi MaGHV tăng dần.
- Số giữ chỗ = số lịch cùng MaGHV, TrangThai = Đã đặt, DaTruBuoi = 0, gồm cả lịch quá giờ chưa trừ. Đặt/hủy/trừ phải kiểm tra lại trong cùng giao dịch để không giữ hoặc trừ quá số buổi.
- Hủy đúng hạn loại lịch khỏi số giữ chỗ; tác vụ đến giờ giảm SoBuoiConLai và đổi cờ cùng lúc. Không chuyển lịch sang gói khác khi mua thêm hoặc khi chạy tác vụ.

### BR-C16 — PT thuộc đúng chi nhánh

- Tại `NgayGio`, nhân viên phụ trách phải có đúng một giai đoạn công tác hợp lệ tại `MaCN` của lịch.
- Theo #13, LichSuCongTac lưu ChucVu và ngày DATE bao gồm hai biên. Tra ngày địa phương của buổi hẹn, không dùng NhanVien.ChucVu hiện tại. Toàn bộ buổi tập phải nằm từ 00:00 NgayBatDau đến trước 00:00 ngày sau NgayKetThuc (nếu có); buổi kết thúc đúng biên này vẫn thuộc giai đoạn cũ. GD1-06 đã chốt: đối chiếu lịch sử dự kiến khi đổi vị trí; còn lịch chưa hủy không phù hợp thì từ chối toàn bộ thay đổi và trả danh sách lịch vướng. Bao gồm lịch kéo qua mốc hiệu lực; kết thúc đúng mốc không bị ảnh hưởng. Không tự hủy/chuyển lịch; hủy vẫn tuân theo hai giờ.
- Đặt lịch và đổi vị trí phải kiểm tra nhất quán lịch sử/lịch hẹn khi ghi nhận. Sau khi chấp nhận đổi vị trí, mọi lịch đặt mới phải phù hợp cả giai đoạn chờ.
- Điều kiện được kiểm tra qua `LichSuCongTac` của A; `LichHenPT` không lưu khóa ngoại đến một bản ghi lịch sử công tác nên quan hệ này không vẽ trên ERD C.

### BR-C17 — Chuyển trạng thái lịch

- `TrangThai` chỉ thuộc `Đã đặt`, `Đã hoàn thành`, `Hủy`.
- Chỉ cho phép `Đã đặt → Đã hoàn thành` hoặc `Đã đặt → Hủy`.
- Lịch đã hoàn thành hoặc đã hủy không được chuyển trở lại `Đã đặt`.

### BR-C18 — Trừ buổi PT đúng một lần

- `DaTruBuoi` chỉ được chuyển từ 0 sang 1.
- Một lịch chỉ làm giảm quyền lợi PT của chính MaGHV tối đa một lần, kể cả khi yêu cầu bị gửi lại. Khi chạy bù, xét hạn gói tại ngày hẹn, không loại bỏ buổi hợp lệ chỉ vì ngày chạy bù đã quá hạn.
- Tại giờ hẹn, lịch chưa hủy tự trừ đúng một buổi của gói đã giữ và chuyển cờ thành 1, không phụ thuộc check-in hoặc tham dự.
- Chỉ cho hội viên hủy khi còn ít nhất hai giờ: giải phóng buổi giữ, cờ vẫn 0. Hủy muộn bị từ chối. Lịch hoàn thành hoặc vắng mặt phải có cờ 1; ghi nhận kết quả không trừ thêm. Tác vụ đến giờ phải chạy lại an toàn khi lỗi, không bỏ sót lịch quá hạn chưa trừ.

### BR-C19 — Đánh giá HLV

- Chỉ `LichHenPT` có trạng thái `Đã hoàn thành` mới được đánh giá.
- Người gửi đánh giá phải là hội viên của lịch.
- `Diem` là số nguyên từ 1 đến 5.
- `ThoiDiemDanhGia` không được trước thời điểm buổi tập hoàn thành.

### BR-C20 — Bảo toàn dữ liệu lịch sử

- Không xóa vật lý `CheckIn`, lịch đã hoàn thành/hủy hoặc đánh giá đã dùng cho báo cáo và tính hoa hồng.
- Nếu phải điều chỉnh dữ liệu lịch sử, thao tác phải được phân quyền và có khả năng truy vết ở giai đoạn vật lý.
