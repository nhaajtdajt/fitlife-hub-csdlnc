# Bảng phân tích tần suất — Module A

## 1. Nguyên tắc ước lượng

Đề bài chỉ cung cấp quy mô 12 chi nhánh và số liệu tải cụ thể cho check-in; đề không cung cấp số lượng nhân viên hoặc tần suất nghiệp vụ nhân sự. Vì vậy, các con số trong bảng dưới đây là **ước lượng ban đầu của nhóm** để phục vụ phân tích thiết kế, không phải số liệu đo từ hệ thống thật.

Quy ước mức độ theo số lần thực hiện:

- **Cao:** từ 1.000 lần/ngày trở lên.
- **Trung bình:** từ 20 đến dưới 1.000 lần/ngày.
- **Thấp:** dưới 20 lần/ngày hoặc thực hiện theo tuần/tháng.

Mức độ tần suất không đồng nghĩa với chi phí truy vấn. Một chức năng tần suất thấp vẫn có thể là truy vấn nặng, điển hình là tính hoa hồng cuối tháng.

## 2. Bảng tần suất chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất đề xuất | Thời điểm tập trung | Mức độ | Nhận xét thiết kế |
|---|---|---|---|---:|---|---|---|
| A-F01 | Xem danh sách/chi tiết chi nhánh | Tra cứu địa chỉ, giờ hoạt động, sức chứa, dịch vụ và tiện ích | Đọc | ~100 lần/ngày | Trong giờ hoạt động | Trung bình | Dữ liệu nhỏ, được nhiều module dùng chung |
| A-F02 | Cập nhật chi nhánh | Thay đổi thông tin liên hệ, giờ hoạt động, sức chứa hoặc dịch vụ/tiện ích | Ghi | ~4 lần/tháng | Khi có thay đổi vận hành | Thấp | Cần tránh xóa chi nhánh đã được tham chiếu |
| A-F03 | Tiếp nhận nhân viên | Tạo hồ sơ và giai đoạn công tác đầu tiên | Đọc + ghi | ~5 lần/ngày | Giờ hành chính | Thấp | Hồ sơ và lịch sử đầu tiên phải được tạo trong cùng giao dịch |
| A-F04 | Tra cứu nhân viên hiện tại | Tìm theo mã, tên, chức vụ hoặc chi nhánh hiện tại | Đọc | ~200 lần/ngày | Đầu/giữa ca làm việc | Trung bình | Cần xác định nhanh bản ghi công tác đang hiệu lực |
| A-F05 | Cập nhật hồ sơ nhân viên | Sửa thông tin liên hệ, chức vụ, lương hoặc trạng thái | Đọc + ghi | ~20 lần/ngày | Giờ hành chính | Trung bình | Thay đổi có tính hồi cứu cần được lưu lịch sử phù hợp |
| A-F06 | Điều chuyển nhân viên | Đóng giai đoạn công tác cũ và mở giai đoạn mới | Đọc + ghi | ~10 lần/tháng | Giờ hành chính | Thấp | Tuy ít xảy ra nhưng yêu cầu giao dịch nguyên tử và chống chồng khoảng thời gian |
| A-F07 | Tra cứu lịch sử công tác | Xem toàn bộ lịch sử hoặc tìm chi nhánh của nhân viên tại một thời điểm | Đọc | ~100 lần/ngày | Giờ hành chính; tăng khi chốt lương | Trung bình | Điều kiện chính gồm `MaNV` và khoảng thời gian công tác |
| A-F08 | Cập nhật lương cơ bản | Điều chỉnh lương theo quyết định nhân sự | Đọc + ghi | ~1 lần/tháng/nhân viên được điều chỉnh | Kỳ xét lương | Thấp | Nếu cần tính hồi cứu phải có lịch sử lương, không chỉ ghi đè lương hiện tại |
| A-F09 | Xác định chi nhánh của PT tại ngày dạy | Đối chiếu từng buổi PT với lịch sử công tác | Đọc | ~2.600 lần/ngày | Khi xem buổi dạy; dồn khối lượng khi tính hoa hồng cuối tháng | Cao | Theo giả định chung: khoảng 9.000 hội viên dùng PT, trung bình 2 buổi/tuần, tương đương khoảng 2.600 buổi diễn ra mỗi ngày. Đây là truy vấn lõi của vấn đề số 5; khi xử lý cả kỳ, hệ thống phải đối chiếu khoảng 78.000 buổi/tháng |
| A-F10 | Tính hoa hồng PT theo chi nhánh | Tổng hợp lịch hẹn hoàn thành, xác định nơi công tác tại ngày dạy và tính tiền | Đọc nặng | 1 lần/tháng/chi nhánh, tương đương 12 lần/tháng toàn hệ thống | Cuối tháng | Thấp về tần suất, nặng về chi phí | Phải trả kết quả đúng khi PT chuyển chi nhánh giữa tháng |
| A-F11 | Xem/xuất bảng hoa hồng | Xem chi tiết số buổi, chi nhánh và thành tiền của từng PT | Đọc | ~2 lần/tháng/chi nhánh | Cuối và đầu tháng kế tiếp | Thấp | Nên cho phép truy vết từ dòng tổng hợp về các buổi dạy nguồn |

## 3. Nhận xét ban đầu

1. `ChiNhanh` có rất ít dòng nhưng được tham chiếu rộng; ưu tiên tính ổn định của mã và thông tin công bố cho module khác.
2. `LichSuCongTac` là dữ liệu quan trọng nhất của Module A. Các thao tác ghi không nhiều nhưng phải bảo đảm tuyệt đối không có khoảng công tác chồng lấn.
3. Tra cứu theo nhân viên và thời điểm là mẫu truy vấn chính. Đây là cơ sở để phân tích chỉ mục ở giai đoạn vật lý, nhưng chưa chốt chỉ mục tại giai đoạn này.
4. Tính hoa hồng có tần suất thấp nhưng phải đọc nhiều lịch hẹn và thực hiện đối chiếu thời gian cho từng buổi, vì vậy có khả năng trở thành truy vấn nặng.
5. A-F09 được ước lượng khoảng 2.600 lần/ngày theo bộ giả định chung của nhóm: khoảng 9.000 hội viên có gói PT và trung bình 2 buổi/tuần. Khi tính hoa hồng cuối tháng, các lượt đối chiếu được xử lý tập trung với quy mô khoảng 78.000 buổi/tháng, nên đây là truy vấn có tần suất cao và có thể tạo tải đọc lớn dù A-F10 chỉ chạy 12 lần/tháng.
6. Các con số ước lượng phải được cả nhóm rà soát để nhất quán với 8–10 kịch bản sử dụng chung của hệ thống.
