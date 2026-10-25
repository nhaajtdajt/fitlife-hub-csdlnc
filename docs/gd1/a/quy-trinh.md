# Mô tả quy trình — Module A

## 1. Mục tiêu và phạm vi

Module A quản lý thông tin chi nhánh, nhân viên và toàn bộ quá trình nhân viên làm việc hoặc điều chuyển giữa các chi nhánh. Mục tiêu trọng tâm là lưu được lịch sử công tác theo thời gian để hệ thống trả lời chính xác câu hỏi: **tại thời điểm một buổi PT diễn ra, PT thuộc chi nhánh nào?**

Kết quả này được dùng để tổng hợp số buổi dạy và tính hoa hồng PT theo đúng chi nhánh, kể cả khi PT chuyển công tác giữa tháng.

## 2. Tác nhân và dữ liệu liên quan

### 2.1. Tác nhân

| Tác nhân | Trách nhiệm |
|---|---|
| Quản trị hệ thống | Tạo và cập nhật thông tin chi nhánh, dịch vụ và tiện ích |
| Quản lý nhân sự | Tạo hồ sơ nhân viên, cập nhật thông tin, lương và thực hiện điều chuyển |
| Quản lý chi nhánh | Gửi hoặc xác nhận yêu cầu điều chuyển; xem nhân sự và hoa hồng thuộc chi nhánh |
| Kế toán/ban giám đốc | Chốt kỳ, xem và xuất bảng tổng hợp hoa hồng PT |
| Hệ thống | Kiểm tra ràng buộc thời gian, lưu lịch sử công tác và tổng hợp buổi dạy |

### 2.2. Dữ liệu

| Dữ liệu | Nội dung chính |
|---|---|
| `ChiNhanh` | Mã, tên, địa chỉ, số điện thoại, giờ mở/đóng cửa, sức chứa |
| Dịch vụ/tiện ích chi nhánh | Các dịch vụ hoặc tiện ích hiện có tại mỗi chi nhánh |
| `NhanVien` | Mã, họ tên, số điện thoại, chức vụ, lương cơ bản và trạng thái làm việc |
| `LichSuCongTac` | Nhân viên, chi nhánh, thời điểm bắt đầu và thời điểm kết thúc |
| `LichHenPT` | PT, chi nhánh dạy, ngày giờ và trạng thái buổi hẹn; dữ liệu do Module C quản lý |

## 3. Quy tắc nghiệp vụ chung

1. Mỗi nhân viên có một mã duy nhất.
2. Mỗi bản ghi lịch sử công tác phải có thời điểm bắt đầu; thời điểm kết thúc có thể để trống khi giai đoạn công tác còn hiệu lực.
3. Thời điểm kết thúc phải lớn hơn thời điểm bắt đầu.
4. Các khoảng công tác của cùng một nhân viên không được chồng lấn.
5. Tại một thời điểm, một nhân viên chỉ thuộc tối đa một chi nhánh.
6. Không được điều chuyển nhân viên sang chính chi nhánh đang công tác.
7. Không xóa vật lý lịch sử công tác đã được dùng để đối chiếu lịch dạy hoặc tính hoa hồng.
8. Chỉ lịch hẹn PT có trạng thái `Đã hoàn thành` mới được tính hoa hồng.
9. Mỗi buổi dạy được tính vào chi nhánh mà PT thuộc về tại thời điểm buổi dạy diễn ra. Nếu không tìm thấy hoặc tìm thấy nhiều hơn một giai đoạn công tác phù hợp, hệ thống phải báo dữ liệu bất thường thay vì tự tính.
10. **Đề xuất:** biểu diễn giai đoạn công tác theo khoảng nửa mở `[NgayBatDau, NgayKetThuc)`. Bản ghi hiện tại có `NgayKetThuc` rỗng. Cách biểu diễn này giúp thời điểm kết thúc công tác cũ đồng thời là thời điểm bắt đầu công tác mới mà không bị chồng lấn.
11. **Đề xuất:** hoa hồng là mức cố định cho mỗi buổi PT hoàn thành và được phân bổ theo chi nhánh tại ngày dạy. Công thức và mức tiền cụ thể phải được nhóm chốt trước khi thiết kế logic.

## 4. QT-A01 — Quản lý thông tin chi nhánh

### Mục đích

Tạo mới và duy trì thông tin của 12 chi nhánh, bao gồm dịch vụ và tiện ích được cung cấp tại từng nơi.

### Điều kiện trước

- Người thực hiện có quyền quản trị.
- Mã chi nhánh mới chưa tồn tại.

### Luồng chính

1. Quản trị viên chọn tạo mới hoặc chỉnh sửa chi nhánh.
2. Hệ thống yêu cầu nhập tên, địa chỉ, số điện thoại, giờ mở cửa, giờ đóng cửa và sức chứa.
3. Người dùng chọn danh sách dịch vụ/tiện ích có tại chi nhánh.
4. Hệ thống kiểm tra dữ liệu bắt buộc, định dạng và tính hợp lệ của giờ hoạt động.
5. Hệ thống lưu thay đổi và ghi nhận kết quả.

### Ngoại lệ

- Trùng mã chi nhánh: từ chối tạo mới.
- Giờ đóng cửa không hợp lệ so với giờ mở cửa: yêu cầu nhập lại.
- Chi nhánh đã có dữ liệu nhân viên, check-in, tồn kho hoặc hóa đơn: không cho xóa vật lý; chuyển sang trạng thái ngừng hoạt động nếu nhóm chọn hỗ trợ nghiệp vụ này.

### Kết quả

Thông tin chi nhánh và danh sách dịch vụ/tiện ích sẵn sàng cho các module khác tham chiếu.

## 5. QT-A02 — Tiếp nhận nhân viên và mở giai đoạn công tác đầu tiên

### Mục đích

Tạo hồ sơ cho nhân viên mới và xác lập chi nhánh công tác ban đầu.

### Điều kiện trước

- Chi nhánh tiếp nhận đang tồn tại và hoạt động.
- Mã nhân viên chưa tồn tại.
- Chức vụ thuộc danh mục được chấp nhận: PT, lễ tân, quản lý chi nhánh hoặc nhân viên bán hàng.

### Luồng chính

1. Quản lý nhân sự nhập hồ sơ nhân viên, chức vụ, lương cơ bản, chi nhánh tiếp nhận và ngày bắt đầu.
2. Hệ thống kiểm tra dữ liệu bắt buộc và tính duy nhất của mã nhân viên.
3. Hệ thống tạo hồ sơ `NhanVien`.
4. Hệ thống tạo bản ghi `LichSuCongTac` đầu tiên, có ngày bắt đầu và chưa có ngày kết thúc.
5. Hệ thống thông báo tiếp nhận thành công.

### Ngoại lệ

- Mã nhân viên đã tồn tại: từ chối tạo.
- Chi nhánh không tồn tại hoặc đã ngừng hoạt động: từ chối tiếp nhận.
- Không thể tạo lịch sử công tác: toàn bộ thao tác phải được hủy để tránh có hồ sơ nhân viên không xác định được nơi công tác.

### Kết quả

Nhân viên có hồ sơ hợp lệ và đúng một giai đoạn công tác đang hiệu lực.

## 6. QT-A03 — Điều chuyển nhân viên giữa các chi nhánh

### Mục đích

Chuyển nhân viên sang chi nhánh khác mà vẫn bảo toàn lịch sử công tác để phục vụ truy vấn hồi cứu và tính hoa hồng.

### Điều kiện trước

- Nhân viên đang làm việc và có đúng một bản ghi công tác hiện hành.
- Chi nhánh đích tồn tại, đang hoạt động và khác chi nhánh hiện tại.
- Thời điểm hiệu lực điều chuyển phải sau thời điểm bắt đầu giai đoạn hiện hành.

### Luồng chính

1. Quản lý chi nhánh gửi yêu cầu điều chuyển, gồm nhân viên, chi nhánh đích, thời điểm hiệu lực và lý do.
2. Quản lý nhân sự kiểm tra và xác nhận yêu cầu.
3. Hệ thống khóa dữ liệu công tác hiện hành của nhân viên trong thời gian xử lý.
4. Hệ thống kiểm tra chi nhánh đích và kiểm tra khoảng thời gian mới không chồng với lịch sử đã có.
5. Hệ thống cập nhật thời điểm kết thúc của giai đoạn công tác hiện hành bằng thời điểm hiệu lực điều chuyển.
6. Hệ thống tạo giai đoạn công tác mới tại chi nhánh đích, bắt đầu đúng thời điểm hiệu lực và chưa có thời điểm kết thúc.
7. Hệ thống hoàn tất đồng thời hai thay đổi và thông báo thành công.

### Ngoại lệ

- Chi nhánh đích trùng chi nhánh hiện tại: từ chối điều chuyển.
- Thời điểm hiệu lực tạo ra khoảng chồng lấn hoặc khoảng thời gian không hợp lệ: từ chối thao tác.
- Không có hoặc có nhiều bản ghi công tác hiện hành: dừng xử lý và yêu cầu sửa dữ liệu.
- Một trong hai bước đóng bản ghi cũ hoặc mở bản ghi mới thất bại: hoàn tác toàn bộ giao dịch.
- Có hai yêu cầu đồng thời cho cùng nhân viên: chỉ một giao dịch được hoàn tất; giao dịch còn lại phải đọc lại trạng thái mới.

### Kết quả

Lịch sử cũ được giữ nguyên, giai đoạn mới được mở và tại mọi thời điểm nhân viên chỉ thuộc một chi nhánh.

## 7. QT-A04 — Cập nhật thông tin nhân viên và lương cơ bản

### Mục đích

Duy trì thông tin liên hệ, chức vụ và lương cơ bản của nhân viên.

### Luồng chính

1. Quản lý nhân sự tìm nhân viên theo mã, tên hoặc chi nhánh hiện tại.
2. Hệ thống hiển thị hồ sơ và nơi công tác hiện hành.
3. Người dùng chỉnh sửa các thông tin được phép.
4. Hệ thống kiểm tra định dạng và quyền cập nhật.
5. Hệ thống lưu thay đổi.

### Ngoại lệ

- Nhân viên không tồn tại hoặc đã nghỉ việc: không cho cập nhật dữ liệu hiện hành không phù hợp.
- Giá trị lương âm hoặc không hợp lệ: từ chối cập nhật.
- Nếu cần tính lương hồi cứu theo thời gian nhưng hệ thống chỉ lưu lương hiện tại, hệ thống phải cảnh báo đây là thiếu hụt dữ liệu; nhóm cần cân nhắc bổ sung lịch sử lương ở bước thiết kế.

### Kết quả

Hồ sơ nhân viên được cập nhật; các thay đổi cần truy vết không làm mất dữ liệu lịch sử.

## 8. QT-A05 — Tra cứu lịch sử công tác

### Mục đích

Tra cứu nhân viên thuộc chi nhánh nào tại một thời điểm hoặc xem toàn bộ quá trình điều chuyển.

### Luồng chính

1. Người dùng nhập nhân viên và có thể nhập thêm thời điểm cần tra cứu.
2. Nếu có thời điểm, hệ thống tìm giai đoạn thỏa `NgayBatDau <= ThoiDiem` và `NgayKetThuc` rỗng hoặc `ThoiDiem < NgayKetThuc`.
3. Nếu không có thời điểm, hệ thống trả về toàn bộ lịch sử theo thứ tự thời gian.
4. Hệ thống hiển thị chi nhánh, thời gian bắt đầu, thời gian kết thúc và thông tin liên quan.

### Ngoại lệ

- Không có giai đoạn phù hợp: thông báo thiếu lịch sử tại thời điểm yêu cầu.
- Có nhiều giai đoạn phù hợp: đánh dấu dữ liệu chồng lấn và không tự chọn một kết quả.

### Kết quả

Người dùng nhận được lịch sử đầy đủ hoặc đúng một chi nhánh tại thời điểm cần tra cứu.

## 9. QT-A06 — Tính hoa hồng PT theo kỳ

### Mục đích

Tổng hợp đúng số buổi dạy của PT cho từng chi nhánh trong một kỳ, kể cả khi PT điều chuyển giữa kỳ.

### Điều kiện trước

- Kỳ tính hoa hồng đã được chọn.
- Dữ liệu lịch hẹn PT của Module C đã cập nhật trạng thái hoàn thành.
- Lịch sử công tác của PT không bị chồng lấn.
- Công thức và mức hoa hồng đã được nhóm phê duyệt.

### Luồng chính

1. Kế toán hoặc quản lý chọn kỳ tính, có thể lọc theo PT hoặc chi nhánh.
2. Hệ thống lấy các lịch hẹn PT có trạng thái `Đã hoàn thành` và thời gian nằm trong kỳ.
3. Với mỗi buổi dạy, hệ thống đối chiếu thời gian dạy với `LichSuCongTac` để tìm chi nhánh mà PT thuộc về tại thời điểm đó.
4. Hệ thống kiểm tra chi nhánh của lịch hẹn với kết quả đối chiếu theo quy tắc mà nhóm đã chốt.
5. Hệ thống nhóm các buổi hợp lệ theo PT và chi nhánh.
6. Hệ thống tính hoa hồng theo công thức đã cấu hình.
7. Hệ thống hiển thị tổng số buổi, mức hoa hồng, thành tiền và các dòng dữ liệu bất thường.
8. Người có thẩm quyền xác nhận hoặc xuất bảng tổng hợp.

### Ngoại lệ

- Buổi dạy không tìm được giai đoạn công tác tương ứng: không tính tự động và đưa vào danh sách cần xử lý.
- Buổi dạy khớp nhiều giai đoạn: báo lỗi chồng lấn lịch sử công tác.
- Chi nhánh ghi trên lịch hẹn khác chi nhánh xác định từ lịch sử công tác: đánh dấu sai lệch để nhóm xác định quy tắc xử lý.
- Lịch hẹn chưa hoàn thành hoặc đã hủy: không tính hoa hồng.
- Kỳ đã chốt: không cho tính lại nếu chưa thực hiện quy trình mở khóa hoặc điều chỉnh.

### Kết quả

Bảng hoa hồng phản ánh số buổi dạy theo đúng chi nhánh mà PT thuộc về tại thời điểm từng buổi diễn ra.

## 10. Quan hệ với module khác

| Module | Dữ liệu Module A cung cấp | Dữ liệu Module A sử dụng |
|---|---|---|
| B — Hội viên và gói | `ChiNhanh`, `NhanVien` khi phát sinh khóa ngoại liên quan | Không bắt buộc cho các quy trình chính của A |
| C — Check-in và lịch PT | `ChiNhanh(MaCN, SucChua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, NgayBatDau, NgayKetThuc)` | `LichHenPT(MaNV, MaCN, NgayGio, TrangThai)` để tính hoa hồng |
| D — Hóa đơn và báo cáo | `ChiNhanh` và `NhanVien` cho tồn kho, hóa đơn và báo cáo | Không bắt buộc cho quy trình điều chuyển |
