# Module C — Mô tả quy trình nghiệp vụ

Phạm vi: cửa kiểm soát, phiên vào–ra, lịch hẹn PT và đánh giá HLV. Module C dùng dữ liệu công bố của A (`ChiNhanh`, `NhanVien`, `LichSuCongTac`) và B (`HoiVien`, `GoiDichVu`, `GoiHoiVien`), đồng thời công bố dữ liệu lịch hẹn cho A và check-in cho D theo [hợp đồng giao diện](../../phan-cong.md).

Số lượng lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Quyết định #1, #2, #7 và #8 đã chốt. Các quy tắc một giờ, hủy trước hai giờ, PT chỉ dạy tại nơi đang công tác và tự đóng lượt lúc chi nhánh đóng cửa lấy từ đề xuất #10–#12, #15, hiện còn cần nhóm xác nhận.

## Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Hội viên | Quét mã vào/ra, xem lịch trống, đặt/hủy lịch và đánh giá buổi PT |
| Lễ tân | Hỗ trợ xử lý lượt quét lỗi, tra cứu người đang có mặt và lịch hẹn |
| PT | Xem lịch dạy; xác nhận kết quả buổi tập theo quyền được phân công |
| Quản lý chi nhánh | Quản lý các cửa kiểm soát, theo dõi công suất và xử lý dữ liệu bất thường |
| Hệ thống | Xác thực quyền vào, ghi log, chống lịch trùng, trừ quyền lợi đúng một lần và tự đóng lượt |

| Dữ liệu | Nội dung |
|---|---|
| `CuaKiemSoat` | Chi nhánh, số hiệu cửa, chức năng quét và trạng thái hoạt động; khóa ghép `(MaCN, SoHieuCua)` |
| `CheckIn` | Hội viên, chi nhánh, số cửa vào/ra và thời gian vào/ra |
| `LichHenPT` | Hội viên, PT, chi nhánh, giờ bắt đầu, trạng thái và cờ đã trừ buổi |
| `DanhGiaHLV` | Một điểm số/nhận xét cho một lịch PT đã hoàn thành |
| Dữ liệu A | Chi nhánh, chức vụ nhân viên và khoảng công tác để xác nhận PT thuộc đúng chi nhánh tại giờ dạy |
| Dữ liệu B | Hội viên, loại gói, hạn và số buổi còn lại để xác định quyền vào/đặt lịch |

## Quy tắc nghiệp vụ chung

1. Mỗi hội viên chỉ có tối đa một `CheckIn` chưa có `TGRa` trên toàn hệ thống. `TGRa` phải sau `TGVao`.
2. Cửa quét phải đang hoạt động, thuộc cùng chi nhánh với lượt check-in và cho phép đúng chiều quét. Dữ liệu lịch sử không bị xóa khi cửa ngừng hoạt động.
3. Hội viên được vào bằng một trong các quyền: Gym/Yoga còn hạn; vé Spa còn lượt; hoặc gói PT còn hạn/còn buổi và có lịch PT hợp lệ trong ngày. Gói dùng được tại mọi chi nhánh.
4. Nếu có Gym/Yoga hợp lệ thì không trừ vé Spa tại cổng. Nếu dùng quyền Spa tại cổng thì trừ đúng một lượt. Nếu có lịch PT hợp lệ thì buổi PT được trừ tối đa một lần cho lịch đó.
5. Giao dịch quét vào phải nguyên tử: xác thực quyền lợi, trừ quyền lợi nếu cần, đánh dấu lịch đã trừ và tạo `CheckIn` cùng thành công hoặc cùng thất bại.
6. Lịch PT kéo dài một giờ theo khung cấu hình. Một PT không có hai lịch chưa hủy chồng nhau; với khung cố định, cặp `(MaNV, NgayGio)` của lịch chưa hủy là duy nhất.
7. Nhân viên trong lịch phải có chức vụ PT và có đúng một giai đoạn công tác tại chi nhánh lịch hẹn trong toàn bộ buổi tập.
8. Khi đặt lịch, hội viên phải có gói PT còn hạn/còn buổi và Gym/Yoga còn hạn tại ngày hẹn. Số lịch chưa hủy chưa tiêu thụ không được vượt số buổi PT còn khả dụng.
9. Trạng thái lịch chỉ gồm `Đã đặt`, `Đã hoàn thành`, `Hủy`. Chuyển trạng thái phải theo luồng cho phép; lịch hoàn thành hoặc đã hủy không được đưa lại về `Đã đặt`.
10. Hủy trước ít nhất hai giờ không mất buổi. Hủy muộn hoặc không đến làm mất một buổi nhưng không được tính hoa hồng; `DaTruBuoi` ngăn trừ lặp. Quy tắc này đang chờ nhóm chốt.
11. Chỉ lịch `Đã hoàn thành` mới được đánh giá và tính hoa hồng. Mỗi lịch có tối đa một đánh giá, điểm nguyên từ 1 đến 5.
12. Lượt quên quét ra được tự đóng tại giờ đóng cửa của chi nhánh và đánh dấu `TuDong`; không được sửa làm mất dấu dữ liệu gốc. Quy tắc này đang chờ nhóm chốt.

## QT-C1. Quét vào và xác thực quyền vào cổng

**Ai làm:** hội viên tại cửa vào; lễ tân chỉ hỗ trợ khi có lỗi.

**Điều kiện trước:** cửa đang hoạt động, có chức năng `Vào` hoặc `Hai chiều`, nhận diện được chi nhánh và mã hội viên.

1. Cửa kiểm soát gửi mã hội viên, `MaCN`, `SoHieuCua` và thời điểm quét.
2. Hệ thống xác nhận hội viên tồn tại và chưa có lượt check-in đang mở.
3. Hệ thống đọc các gói có thể dùng: Gym/Yoga còn hạn; vé Spa còn lượt; gói PT còn hạn/còn buổi kèm lịch `Đã đặt` trong ngày.
4. Hệ thống chọn quyền theo thứ tự nghiệp vụ: Gym/Yoga cho quyền vào chung; PT được gắn với lịch hợp lệ cần tiêu thụ; chỉ dùng Spa tại cổng khi không có Gym/Yoga/PT đủ điều kiện.
5. Nếu dùng PT hoặc Spa, hệ thống khóa đúng quyền lợi cần cập nhật và kiểm tra lại số buổi/lượt. Với PT, hệ thống cũng khóa lịch và kiểm tra `DaTruBuoi = 0`.
6. Trong một giao dịch, hệ thống giảm quyền lợi nếu cần, đặt `DaTruBuoi = 1` cho lịch PT tương ứng và tạo một dòng `CheckIn` có `TGVao`, chưa có `TGRa`.
7. Cửa kiểm soát mở cổng và trả kết quả. Đường xử lý mục tiêu dưới 50 ms; báo cáo/tổng hợp không chạy đồng bộ trong giao dịch này.

**Ngoại lệ:** mã không hợp lệ; cửa không tồn tại, ngừng hoạt động hoặc không cho phép chiều vào; đã có phiên đang mở; không có quyền hợp lệ; gói vừa hết lượt do giao dịch khác; không xác định duy nhất lịch PT cần trừ; chi nhánh đã đủ sức chứa nếu nhóm áp dụng chặn công suất. Mọi lỗi đều không mở cổng, không tạo log và không trừ quyền lợi một phần.

## QT-C2. Quét ra

**Ai làm:** hội viên tại cửa ra.

1. Cửa kiểm soát gửi mã hội viên, `MaCN`, `SoHieuCua` và thời điểm quét.
2. Hệ thống tìm đúng một `CheckIn` đang mở của hội viên.
3. Hệ thống kiểm tra cửa thuộc cùng chi nhánh, cho phép chiều ra và thời gian ra sau thời gian vào.
4. Hệ thống cập nhật `TGRa`, `SoHieuCuaRa`, rồi mở cổng.

**Ngoại lệ:** không có hoặc có nhiều lượt đang mở; cửa sai chi nhánh, ngừng hoạt động hoặc không cho phép chiều ra; thời gian không hợp lệ. Hệ thống không tự chọn một dòng khi dữ liệu mâu thuẫn mà chuyển cho lễ tân xử lý.

## QT-C3. Tự đóng lượt quên quét ra

**Ai làm:** tác vụ hệ thống theo từng chi nhánh.

1. Khi chi nhánh kết thúc giờ hoạt động, tác vụ lấy các lượt còn mở có `TGVao` thuộc ngày hoạt động cần đóng.
2. Hệ thống đặt `TGRa` bằng thời điểm đóng cửa áp dụng cho ngày đó; `SoHieuCuaRa` giữ rỗng vì không phát sinh lượt quét tại cửa ra.
3. Hệ thống ghi số dòng đã đóng và danh sách bất thường để quản lý đối chiếu.

Tác vụ phải chạy lặp an toàn: dòng đã đóng không được cập nhật lần hai. Nếu tác vụ thất bại, lần chạy lại chỉ xử lý dòng còn mở. Cách xử lý chi nhánh hoạt động qua nửa đêm cần được bổ sung nếu giờ mở/đóng cho phép trường hợp này.

## QT-C4. Xem lịch trống và đặt lịch PT

**Ai làm:** hội viên hoặc lễ tân thao tác thay hội viên.

1. Người dùng chọn ngày, chi nhánh và có thể chọn PT.
2. Hệ thống lấy các PT có chức vụ phù hợp và giai đoạn công tác bao phủ toàn bộ các khung một giờ tại chi nhánh đó.
3. Hệ thống loại các khung đã có lịch chưa hủy và trả danh sách còn trống.
4. Hội viên chọn PT/khung giờ. Hệ thống kiểm tra tại ngày hẹn: có Gym/Yoga còn hạn, có gói PT còn hạn/còn buổi, và số lịch đang giữ chỗ chưa vượt quyền lợi.
5. Trong giao dịch đặt lịch, hệ thống kiểm tra lại nơi công tác, quyền lợi và khung giờ, sau đó tạo `LichHenPT` trạng thái `Đã đặt`, `DaTruBuoi = 0`.
6. Ràng buộc duy nhất tại CSDL quyết định kết quả cuối cùng khi hai giao dịch cùng chọn một PT/khung. Chỉ một giao dịch thành công; giao dịch còn lại nhận thông báo khung vừa được đặt và phải tải lại lịch.

**Ngoại lệ:** PT không còn làm tại chi nhánh vào ngày hẹn; gói không đủ hạn/buổi; số buổi đã bị các lịch khác giữ chỗ; khung đã bị đặt; ngày giờ ở quá khứ. Không tạo lịch một phần hoặc tự đổi PT/khung.

## QT-C5. Hủy lịch, xử lý không đến và hoàn thành buổi PT

### Hủy lịch

1. Hội viên chọn một lịch `Đã đặt` và yêu cầu hủy.
2. Hệ thống so thời điểm yêu cầu với `NgayGio`.
3. Nếu còn ít nhất hai giờ, hệ thống chuyển sang `Hủy` và không trừ buổi.
4. Nếu hủy muộn, hệ thống khóa lịch và quyền lợi; nếu `DaTruBuoi = 0` thì trừ một buổi và đặt cờ bằng 1, sau đó chuyển trạng thái `Hủy`.

### Không đến

Sau khi khung giờ kết thúc, tác vụ tìm lịch vẫn `Đã đặt` mà không có dấu hiệu tham dự. Hệ thống xử lý như hủy muộn: trừ tối đa một buổi, chuyển `Hủy`, nhưng không tính hoa hồng. Vì tập trạng thái của đề không có `Không đến`, nguyên nhân được lưu ở thuộc tính `LyDoKetThuc` để phân biệt với hủy thường.

### Hoàn thành

Khi buổi tập kết thúc, hệ thống/PT có thẩm quyền chuyển lịch từ `Đã đặt` sang `Đã hoàn thành`. Nếu buổi chưa được trừ tại lúc check-in, giao dịch phải trừ đúng một buổi trước khi hoàn tất. Lịch đã hủy không thể hoàn thành; lịch hoàn thành mới được A tính hoa hồng.

**Ngoại lệ chung:** lịch không tồn tại/sai chủ sở hữu; trạng thái không cho phép; yêu cầu gửi lại; quyền lợi không còn nhưng lịch chưa được đánh dấu đã trừ. Trường hợp dữ liệu lệch phải được báo đối soát, không trừ âm hoặc trừ hai lần.

## QT-C6. Đánh giá HLV

**Ai làm:** hội viên của buổi PT.

1. Hội viên chọn lịch đã hoàn thành chưa có đánh giá.
2. Hội viên nhập điểm nguyên 1–5 và nhận xét tùy chọn.
3. Hệ thống kiểm tra lịch thuộc hội viên, trạng thái `Đã hoàn thành` và chưa được đánh giá.
4. Hệ thống lưu đánh giá gắn duy nhất với lịch; đánh giá được dùng trong báo cáo hiệu suất HLV.

**Ngoại lệ:** lịch chưa hoàn thành/đã hủy, người gửi không phải hội viên của lịch, điểm ngoài miền hoặc lịch đã có đánh giá.

## Quan hệ với module khác

| Module | C cung cấp | C sử dụng / cần xác nhận |
|---|---|---|
| A — Chi nhánh & Nhân sự | `LichHenPT(MaNV, MaCN, NgayGio, TrangThai)` cho tính hoa hồng | `ChiNhanh(MaCN, SucChua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, NgayBatDau, NgayKetThuc)` |
| B — Hội viên, Gói & Thân thiết | Kết quả tham dự để trừ quyền lợi theo quy trình đã thống nhất | `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi)`, `GoiHoiVien(MaHV, MaGoi, NgayHetHan, SoBuoiConLai)`; cần B công bố khóa/thủ tục cập nhật nguyên tử |
| D — Hóa đơn & Báo cáo | `CheckIn(MaHV, MaCN, TGVao, TGRa)` cho tỷ lệ lấp đầy | Không ghi bảng D |
