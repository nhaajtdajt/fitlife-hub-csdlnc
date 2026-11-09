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
| `TienIch`, `CungCapTienIch` | Danh mục dịch vụ/tiện ích và các khoảng ngày từng chi nhánh cung cấp |
| `NhanVien` | Mã, họ tên, số điện thoại, chức vụ, lương cơ bản và trạng thái làm việc |
| `LichSuCongTac` | Nhân viên, chi nhánh, chức vụ tại giai đoạn, ngày bắt đầu và ngày kết thúc (tính cả hai ngày) |
| `LichHenPT` | PT, chi nhánh dạy, ngày giờ và trạng thái buổi hẹn; dữ liệu do Module C quản lý |
| `KyHoaHong`, `ChiTietHoaHong` | Mức hoa hồng và danh sách buổi PT được giữ cố định khi chốt kỳ |

## 3. Quy tắc nghiệp vụ chung

1. Mỗi nhân viên có một mã duy nhất.
2. Mỗi bản ghi lịch sử công tác lưu NgayBatDau, NgayKetThuc kiểu DATE và ChucVu tại giai đoạn đó. NgayKetThuc rỗng nghĩa là chưa xác định ngày kết thúc, không đồng nghĩa bản ghi đang có hiệu lực hôm nay.
3. NgayKetThuc nếu có phải >= NgayBatDau; cho phép giai đoạn kéo dài một ngày. Khi đổi vị trí, NgayBatDau mới = NgayKetThuc cũ + 1 ngày.
4. Các khoảng công tác của cùng một nhân viên không được chồng lấn.
5. Tại một thời điểm, một nhân viên chỉ thuộc tối đa một chi nhánh.
6. Không được điều chuyển nhân viên sang chính chi nhánh đang công tác.
7. Không xóa vật lý lịch sử công tác đã được dùng để đối chiếu lịch dạy hoặc tính hoa hồng.
8. Chỉ lịch hẹn PT có trạng thái `Đã hoàn thành` mới được tính hoa hồng.
9. Mỗi buổi dạy được tính vào chi nhánh mà PT thuộc về tại thời điểm buổi dạy diễn ra. Nếu không tìm thấy hoặc tìm thấy nhiều hơn một giai đoạn công tác phù hợp, hệ thống phải báo dữ liệu bất thường thay vì tự tính.
10. **Đã chốt:** khoảng công tác bao gồm cả hai ngày `[NgayBatDau, NgayKetThuc]`. Bấm đổi chức vụ/chi nhánh trong ngày D thì giai đoạn cũ kết thúc hết D, giai đoạn mới bắt đầu D+1 theo múi giờ Asia/Saigon. Không áp dụng giữa ngày hoặc hồi tố trong luồng này. Tìm giai đoạn hiện hành bằng ngày hôm nay nằm trong khoảng, không chỉ bằng NgayKetThuc rỗng.
11. **Đã chốt:** hoa hồng được tính theo số buổi PT hoàn thành hợp lệ và mức `HoaHongMoiBuoi` của chi nhánh tại lúc chốt kỳ. Hệ thống lưu mức này trong `KyHoaHong` và lưu danh sách buổi trong `ChiTietHoaHong`; thay đổi mức hiện tại không làm thay đổi kỳ đã chốt.
12. Nhân viên mới trong phạm vi hiện tại được tiếp nhận có hiệu lực ngay trong ngày tạo hồ sơ. Nghỉ việc hoặc thay đổi vị trí được gửi trong ngày D có hiệu lực từ D+1; không hồi tố hoặc đổi giữa ngày.
13. Chi nhánh hiện tại của nhân viên được suy ra từ đúng một `LichSuCongTac` bao phủ ngày tra cứu. `NhanVien` chỉ lưu `ChucVu` hiện tại để phục vụ truy cập nhanh và phải được đồng bộ từ lịch sử khi sang ngày mới.

## 4. QT-A01 — Quản lý thông tin chi nhánh

### Mục đích

Tạo mới và duy trì thông tin của 12 chi nhánh, bao gồm dịch vụ và tiện ích được cung cấp tại từng nơi.

### Điều kiện trước

- Người thực hiện có quyền quản trị.
- Mã chi nhánh mới chưa tồn tại.

### Luồng chính

1. Quản trị viên chọn tạo mới hoặc chỉnh sửa chi nhánh.
2. Hệ thống yêu cầu nhập tên, địa chỉ, số điện thoại, giờ mở cửa, giờ đóng cửa và sức chứa.
3. Người dùng chọn danh sách dịch vụ/tiện ích dự kiến cung cấp tại chi nhánh. Việc mở, ngừng hoặc cung cấp lại theo ngày được thực hiện theo QT-A07, không ghi đè lịch sử cung cấp.
4. Hệ thống kiểm tra dữ liệu bắt buộc, SucChua là số nguyên dương và giờ mở < giờ đóng trong cùng ngày Asia/Saigon theo quy ước mô phỏng #15. SucChua chỉ phục vụ báo cáo, không giới hạn lượt vào. Giờ hoạt động/sức chứa cố định trong kỳ dữ liệu mô phỏng; thay đổi cấu hình không được dùng để diễn giải lại lịch sử đã báo cáo.
5. Hệ thống lưu thay đổi và ghi nhận kết quả.

### Ngoại lệ

- Trùng mã chi nhánh: từ chối tạo mới.
- Giờ đóng cửa không hợp lệ so với giờ mở cửa, hoặc sức chứa không phải số nguyên dương: yêu cầu nhập lại.
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

1. Quản lý nhân sự nhập hồ sơ nhân viên, chức vụ, lương cơ bản và chi nhánh tiếp nhận. Ngày bắt đầu được hệ thống lấy là ngày hiện tại theo Asia/Saigon.
2. Hệ thống kiểm tra dữ liệu bắt buộc, tính duy nhất của mã nhân viên, trạng thái hoạt động của chi nhánh và xác nhận ngày bắt đầu là ngày hiện tại.
3. Hệ thống tạo hồ sơ `NhanVien`.
4. Hệ thống tạo bản ghi `LichSuCongTac` đầu tiên, có ngày bắt đầu, chức vụ trùng hồ sơ và chưa có ngày kết thúc; hồ sơ và lịch sử cùng thành công hoặc cùng hoàn tác.
5. Hệ thống thông báo tiếp nhận thành công.

### Ngoại lệ

- Mã nhân viên đã tồn tại: từ chối tạo.
- Chi nhánh không tồn tại hoặc đã ngừng hoạt động: từ chối tiếp nhận.
- Ngày bắt đầu khác ngày hiện tại: từ chối luồng này; hồ sơ có ngày bắt đầu tương lai chưa thuộc phạm vi hiện tại.
- Không thể tạo lịch sử công tác: toàn bộ thao tác phải được hủy để tránh có hồ sơ nhân viên không xác định được nơi công tác.

### Kết quả

Nhân viên có hồ sơ hợp lệ và đúng một giai đoạn công tác đang hiệu lực.

## 6. QT-A03 — Điều chuyển nhân viên giữa các chi nhánh

### Mục đích

Chuyển nhân viên sang chi nhánh khác mà vẫn bảo toàn lịch sử công tác để phục vụ truy vấn hồi cứu và tính hoa hồng.

### Điều kiện trước

- Nhân viên đang làm việc và có đúng một giai đoạn bao phủ ngày hôm nay.
- Chi nhánh đích tồn tại, đang hoạt động và khác chi nhánh hiện tại.
- Chưa có thay đổi vị trí đang chờ hiệu lực của nhân viên. Nếu đã có, từ chối yêu cầu tiếp theo và hiển thị thay đổi đang chờ; không ghi đè hoặc tạo hai giai đoạn cùng bắt đầu ngày mai.

### Luồng chính

1. Người có quyền xác nhận nhân viên, chi nhánh đích và lý do; có thể đổi kèm chức vụ trong cùng yêu cầu. Nếu không đổi chức vụ, giữ chức vụ của giai đoạn hiện hành.
2. Trong giao dịch, kiểm tra lại lịch sử và ngày hiện tại D theo Asia/Saigon. Ngày hiệu lực do hệ thống tính là D+1; hiển thị rõ ngày này khi thông báo kết quả.
3. Kiểm tra lại quyền cập nhật, trạng thái nhân viên, chi nhánh đích và không có giai đoạn chờ. Dòng có NgayKetThuc rỗng nhưng bắt đầu ngày mai không được coi là hiện hành hôm nay.
4. Trước khi ghi, đối chiếu các lịch PT theo mục Kiểm tra lịch PT dưới đây với lịch sử dự kiến sau thay đổi; có lịch không phù hợp thì từ chối toàn bộ yêu cầu. Nếu đều phù hợp, đặt NgayKetThuc của giai đoạn cũ = D. Tạo giai đoạn mới có NgayBatDau = D+1, NgayKetThuc rỗng, chi nhánh/chức vụ mới. Kiểm tra không chồng khoảng trên cả hai biên ngày.
5. Ghi hai thay đổi trong cùng giao dịch. Giữ `NhanVien.ChucVu` theo giai đoạn của D; không đổi sang chức vụ ngày mai ngay khi bấm. Chi nhánh hiện tại không lưu lặp trên `NhanVien`, mà được suy ra từ lịch sử.
6. Khi sang ngày mới, đồng bộ dữ liệu hiện tại theo QT-A04. Lịch sử là nguồn xác định vị trí tại từng ngày.

### Ngoại lệ

- Chi nhánh đích trùng hiện tại: không dùng luồng điều chuyển; đổi chức vụ trong cùng chi nhánh dùng QT-A04.
- Thiếu/trùng giai đoạn hiện hành, khoảng không hợp lệ, hoặc đã có thay đổi đang chờ: từ chối và không ghi dữ liệu một phần.
- Hai yêu cầu đồng thời: chỉ chấp nhận một yêu cầu; yêu cầu sau bị kiểm tra lại và bị từ chối nếu yêu cầu trước đã tạo giai đoạn chờ.
- Bất kỳ bước ghi nào lỗi: hoàn tác cả việc kết thúc giai đoạn cũ và tạo giai đoạn mới.

### Kết quả

Ngày D vẫn thuộc vị trí cũ; từ D+1 thuộc vị trí mới. Ví dụ bấm đổi ngày 15/11 thì kết thúc vị trí cũ ngày 15/11, bắt đầu vị trí mới ngày 16/11. Chỉ tiếp nhận thay đổi khi không làm lịch PT chưa hủy mất điều kiện chức vụ/chi nhánh theo GD1-06.

### Kiểm tra lịch PT trước khi đổi vị trí (GD1-06)

- Gọi E là 00:00 ngày D+1 theo Asia/Saigon. Kiểm tra lại các lịch của nhân viên có TrangThai khác Hủy và thời điểm kết thúc > E. Bao gồm lịch bắt đầu trước E nhưng còn kéo dài sau E; lịch kết thúc đúng E không bị ảnh hưởng.
- Đối chiếu từng lịch với lịch sử dự kiến sau thay đổi: toàn bộ khoảng buổi hẹn phải được một giai đoạn có ChucVu = PT và MaCN trùng lịch bao phủ. Với thiết kế khung một giờ hiện tại, thời điểm kết thúc = NgayGio + 1 giờ; QD10 đã chốt bắt đầu vào giờ tròn (phút/giây bằng 0).
- Có lịch không phù hợp: trả MaLichHen, ngày giờ, chi nhánh và lý do (không còn là PT, sai chi nhánh hoặc không được một giai đoạn bao phủ). Không ghi ngày kết thúc cũ, giai đoạn mới hoặc chức vụ hiện tại. Các lịch và số buổi gói giữ nguyên.
- Không tự hủy/đổi HLV/đổi chi nhánh lịch. Người dùng giải quyết lịch bằng luồng đã được phép rồi gửi lại yêu cầu đổi vị trí; hủy vẫn phải trước ít nhất hai giờ. Nếu không thể hủy hợp lệ thì giữ vị trí cũ để thực hiện lịch, chưa cho đổi. Ngày hiệu lực tính lại từ ngày gửi lại thành công.
- Sau khi đã chấp nhận đổi vị trí, C phải xét cả giai đoạn chờ khi đặt lịch: từ ngày hiệu lực chỉ nhận lịch phù hợp vị trí mới. Buổi đã dạy trước E giữ chức vụ/chi nhánh lịch sử để tính hoa hồng.
- Chống xung đột: đặt lịch thành công trước thì đổi vị trí phải thấy lịch đó; đổi vị trí thành công trước thì đặt lịch phải phù hợp giai đoạn mới. Không dùng kết quả kiểm tra cũ để ghi nhận; lỗi phải giữ nguyên dữ liệu.

## 7. QT-A04 — Cập nhật thông tin nhân viên và lương cơ bản

### Mục đích

Duy trì thông tin liên hệ, chức vụ và lương cơ bản của nhân viên.

### Luồng chính

1. Quản lý nhân sự tìm nhân viên theo mã, tên hoặc chi nhánh hiện tại.
2. Hệ thống hiển thị hồ sơ và nơi công tác hiện hành.
3. Với thông tin liên hệ/lương, kiểm tra định dạng và quyền cập nhật rồi lưu theo luồng hiện có.
4. Với đổi chức vụ, dùng cùng giao dịch đóng/mở giai đoạn như QT-A03, giữ chi nhánh nếu không điều chuyển: kết thúc giai đoạn cũ hết hôm nay, tạo giai đoạn mới từ ngày mai. Không sửa ChucVu của bản ghi lịch sử cũ, không ghi đè chức vụ hiện tại ngay lúc gửi yêu cầu. Từ chối yêu cầu không đổi chức vụ/chi nhánh, đã có thay đổi đang chờ, hoặc còn lịch PT không phù hợp theo kiểm tra ở QT-A03.

### Đồng bộ chức vụ hiện tại khi sang ngày mới

- Khi sang ngày mới theo Asia/Saigon, đồng bộ `NhanVien.ChucVu` từ giai đoạn bao phủ ngày đó. Chi nhánh hiện tại luôn được suy ra từ chính giai đoạn này, không lưu lặp trong `NhanVien`; chạy lại không tạo thêm lịch sử.
- Nếu tác vụ chậm/lỗi, thao tác cần vị trí hiện tại phải đọc giai đoạn theo ngày và đồng bộ trước khi dùng giá trị lưu trên NhanVien; không dùng chức vụ cũ để phân quyền hoặc chọn PT. Tác vụ chạy bù theo ngày thực tế, không kích hoạt mù giai đoạn của ngày đã bỏ lỡ.
- Không tìm được hoặc tìm thấy nhiều giai đoạn: báo dữ liệu bất thường, không tự chọn. Trong ngày đã gửi yêu cầu, giai đoạn có ngày kết thúc hôm nay vẫn là hiện hành.

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
2. Nếu có thời điểm, đổi sang ngày địa phương Asia/Saigon rồi tìm giai đoạn thỏa `NgayBatDau <= NgayTraCuu AND (NgayKetThuc IS NULL OR NgayTraCuu <= NgayKetThuc)`. Dùng cùng điều kiện cho hiện tại và lịch sử.
3. Nếu không có thời điểm, hệ thống trả về toàn bộ lịch sử theo thứ tự thời gian.
4. Hệ thống hiển thị chi nhánh, chức vụ tại giai đoạn, ngày bắt đầu, ngày kết thúc và thông tin liên quan.

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
3. Với mỗi buổi dạy, lấy ngày địa phương của NgayGio, tìm đúng giai đoạn LichSuCongTac bao gồm ngày đó và có ChucVu = PT. Dùng chức vụ/chi nhánh lịch sử, không dùng chức vụ hiện tại để loại buổi dạy cũ.
4. Hệ thống kiểm tra chi nhánh của lịch hẹn với kết quả đối chiếu theo quy tắc mà nhóm đã chốt.
5. Hệ thống nhóm các buổi hợp lệ theo PT và chi nhánh.
6. Kỳ chưa chốt xem trước theo mức HoaHongMoiBuoi của chi nhánh; kỳ đã chốt đọc mức và danh sách buổi đã lưu, không tính lại theo mức hiện tại.
7. Hệ thống hiển thị tổng số buổi, mức hoa hồng, thành tiền và các dòng dữ liệu bất thường.
8. Khi xác nhận chốt tháng đã kết thúc, lưu KyHoaHong và toàn bộ ChiTietHoaHong cùng giao dịch theo [thiết kế logic](thiet-ke-logic.md). Chặn nếu còn lịch Đã đặt hoặc dữ liệu lịch sử sai. Khóa kỳ và đọc lại nguồn/mức trước khi ghi; chạy lại trả kỳ đã có. Xem/xuất kỳ cũ chỉ dùng dữ liệu chốt.

### Ngoại lệ

- Buổi dạy không tìm được giai đoạn công tác tương ứng: không tính tự động và đưa vào danh sách cần xử lý.
- Buổi dạy khớp nhiều giai đoạn: báo lỗi chồng lấn lịch sử công tác.
- Chi nhánh ghi trên lịch hẹn khác chi nhánh xác định từ lịch sử công tác: đánh dấu sai lệch để nhóm xác định quy tắc xử lý.
- Lịch hẹn chưa hoàn thành hoặc đã hủy: không tính hoa hồng.
- Kỳ đã chốt: không mở lại, sửa, xóa hoặc bổ sung buổi trong phạm vi hiện tại. Sửa mức hiện tại không đổi kỳ cũ. C không được bổ sung hoặc sửa nguồn làm thay đổi kỳ đã chốt, kể cả khi cập nhật đồng thời với chốt kỳ.

### Kết quả

Bảng hoa hồng phản ánh số buổi dạy theo đúng chi nhánh mà PT thuộc về tại thời điểm từng buổi diễn ra.

## 10. QT-A07 — Quản lý thời gian cung cấp tiện ích tại chi nhánh

### Mục đích

Ghi nhận một chi nhánh bắt đầu, ngừng hoặc cung cấp lại dịch vụ/tiện ích mà không làm mất lịch sử các giai đoạn trước.

### Điều kiện trước

- Chi nhánh và tiện ích đã tồn tại.
- Người thực hiện có quyền quản trị.
- Ngày hiệu lực thuộc phạm vi dữ liệu được phép cập nhật và không tạo khoảng chồng lấn.

### Luồng chính

1. Quản trị viên chọn chi nhánh, tiện ích và thao tác bắt đầu cung cấp hoặc ngừng cung cấp.
2. Khi bắt đầu cung cấp, hệ thống kiểm tra không có khoảng `CungCapTienIch` nào của cùng cặp chi nhánh–tiện ích bao phủ ngày bắt đầu hoặc giao với khoảng mới.
3. Hệ thống tạo một bản ghi có `NgayBatDau` đã chọn và `NgayKetThuc` rỗng nếu chưa xác định ngày ngừng.
4. Khi ngừng cung cấp, hệ thống tìm đúng một khoảng đang bao phủ ngày kết thúc, kiểm tra `NgayKetThuc >= NgayBatDau` rồi điền ngày kết thúc. Không sửa ngày bắt đầu hoặc xóa khoảng cũ.
5. Khi cung cấp lại sau một lần đã ngừng, hệ thống tạo khoảng mới có ngày bắt đầu sau ngày kết thúc gần nhất; không mở lại bản ghi cũ.
6. Hệ thống lưu thay đổi và hiển thị toàn bộ lịch sử cung cấp theo thứ tự thời gian.

### Ngoại lệ

- Chi nhánh hoặc tiện ích không tồn tại/đã ngừng hoạt động: từ chối bắt đầu cung cấp mới.
- Tiện ích đang được cung cấp tại ngày yêu cầu: không tạo bản ghi trùng.
- Không tìm thấy hoặc tìm thấy nhiều khoảng cần kết thúc: báo dữ liệu bất thường và không tự chọn.
- Khoảng mới chồng lấn hoặc ngày kết thúc trước ngày bắt đầu: từ chối thao tác.

### Kết quả

Mỗi cặp chi nhánh–tiện ích có lịch sử gồm các khoảng ngày không chồng lấn; báo cáo quá khứ không bị diễn giải lại khi cấu hình hiện tại thay đổi.

## 11. QT-A08 — Cho nhân viên nghỉ việc

### Mục đích

Kết thúc quan hệ công tác mà vẫn bảo toàn lịch sử và không để lại lịch PT tương lai không còn người thực hiện hợp lệ.

### Điều kiện trước

- Nhân viên đang làm việc và có đúng một giai đoạn bao phủ ngày hiện tại D.
- Chưa có thay đổi chức vụ/chi nhánh hoặc yêu cầu nghỉ việc khác đang chờ hiệu lực.

### Luồng chính

1. Quản lý nhân sự chọn nhân viên, nhập lý do và xác nhận yêu cầu nghỉ việc trong ngày D; ngày hiệu lực được hệ thống tính là D+1 theo Asia/Saigon.
2. Trong giao dịch, hệ thống kiểm tra lại trạng thái nhân viên, giai đoạn hiện hành và các thay đổi đang chờ.
3. Hệ thống kiểm tra các lịch PT chưa hủy có thời điểm kết thúc sau 00:00 ngày D+1 theo cùng nguyên tắc QT-A03.
4. Nếu không có lịch vướng, hệ thống đặt `NgayKetThuc` của giai đoạn hiện hành bằng D. Trong ngày D, nhân viên vẫn có trạng thái đang làm và chức vụ hiện tại không đổi.
5. Khi sang D+1, hệ thống xác nhận không còn giai đoạn bao phủ ngày mới rồi chuyển `TrangThaiLamViec` sang nghỉ việc. Tác vụ chạy lại phải cho cùng kết quả và không tạo thêm lịch sử.

### Ngoại lệ

- Còn lịch PT chưa hủy sau ngày hiệu lực: từ chối yêu cầu, trả danh sách lịch vướng và không sửa dữ liệu.
- Nhân viên đã nghỉ, không có đúng một giai đoạn hiện hành hoặc đã có thay đổi đang chờ: từ chối yêu cầu.
- Bất kỳ bước ghi nào thất bại: hoàn tác toàn bộ thay đổi.

### Kết quả

Ngày D là ngày làm việc cuối cùng; từ D+1 nhân viên ở trạng thái nghỉ việc, không còn giai đoạn công tác hiện hành và không được chọn cho lịch PT mới. Lịch sử cũ vẫn được giữ để tra cứu và tính hoa hồng kỳ quá khứ.

## 12. Quan hệ với module khác

| Module | Dữ liệu Module A cung cấp | Dữ liệu Module A sử dụng |
|---|---|---|
| B — Hội viên và gói | `ChiNhanh`, `NhanVien` khi phát sinh khóa ngoại liên quan | Không bắt buộc cho các quy trình chính của A |
| C — Check-in và lịch PT | `ChiNhanh(MaCN, SucChua, GioMoCua, GioDongCua)`, `NhanVien(MaNV, ChucVu)`, `LichSuCongTac(MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc)` | `LichHenPT(MaLichHen, MaNV, MaCN, NgayGio, TrangThai)` để tính hoa hồng và chặn đổi vị trí làm lịch không hợp lệ |
| D — Hóa đơn và báo cáo | `ChiNhanh` và `NhanVien` cho tồn kho, hóa đơn và báo cáo | Không bắt buộc cho quy trình điều chuyển |
