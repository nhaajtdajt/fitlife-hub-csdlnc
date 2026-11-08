# Bảng phân tích tần suất — Module A

## 1. Nguyên tắc ước lượng

Đề bài chỉ cung cấp quy mô 12 chi nhánh và số liệu tải cụ thể cho check-in; đề không cung cấp số lượng nhân viên hoặc tần suất nghiệp vụ nhân sự. Vì vậy, các con số trong bảng dưới đây là **ước lượng ban đầu của nhóm** để phục vụ phân tích thiết kế, không phải số liệu đo từ hệ thống thật.

Quy ước mức độ theo số lần thực hiện:

- **Cao:** từ 1.000 lần/ngày trở lên.
- **Trung bình:** từ 100 đến dưới 1.000 lần/ngày.
- **Thấp:** dưới 100 lần/ngày sau quy đổi cùng phạm vi.

Mức độ tần suất không đồng nghĩa với chi phí truy vấn. Một chức năng tần suất thấp vẫn có thể là truy vấn nặng, điển hình là tính hoa hồng cuối tháng.

## 2. Bảng tần suất chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất đề xuất | Thời điểm tập trung | Mức độ | Nhận xét thiết kế |
|---|---|---|---|---:|---|---|---|
| A-F01 | Xem danh sách/chi tiết chi nhánh | Tra cứu địa chỉ, giờ hoạt động, sức chứa, dịch vụ và tiện ích | Đọc | ~100 lần/ngày | Trong giờ hoạt động | Trung bình | Dữ liệu nhỏ, được nhiều module dùng chung |
| A-F02 | Cập nhật chi nhánh | Thay đổi thông tin liên hệ, giờ hoạt động, sức chứa hoặc dịch vụ/tiện ích | Ghi | ~4 lần/tháng | Khi có thay đổi vận hành | Thấp | Cần tránh xóa chi nhánh đã được tham chiếu |
| A-F03 | Tiếp nhận nhân viên | Tạo hồ sơ và giai đoạn công tác đầu tiên | Đọc + ghi | ~5 lần/ngày | Giờ hành chính | Thấp | Hồ sơ và lịch sử đầu tiên phải được tạo trong cùng giao dịch |
| A-F04 | Tra cứu nhân viên hiện tại | Tìm theo mã, tên, chức vụ hoặc chi nhánh hiện tại | Đọc | ~200 lần/ngày | Đầu/giữa ca làm việc | Trung bình | Cần xác định nhanh bản ghi công tác đang hiệu lực |
| A-F05 | Cập nhật hồ sơ nhân viên | Sửa thông tin liên hệ, chức vụ, lương hoặc trạng thái | Đọc + ghi | ~20 lần/ngày | Giờ hành chính | Thấp | Đổi chức vụ kiểm tra lịch PT vướng rồi mới tạo lịch sử từ ngày mai, giữ chức vụ hiện tại hôm nay; phần đổi chức vụ nằm trong số lần cập nhật này |
| A-F06 | Điều chuyển nhân viên | Cũ kết thúc hôm nay, mới bắt đầu ngày mai; ghi hai thay đổi cùng giao dịch | Đọc + ghi | ~10 lần/tháng | Giờ hành chính | Thấp | Đọc lịch PT chưa hủy bị ảnh hưởng, chặn nếu không phù hợp; kiểm tra nhất quán với đặt lịch. Ghi nguyên tử, chống chồng khoảng thời gian |
| A-F07 | Tra cứu lịch sử công tác | Xem toàn bộ lịch sử hoặc tìm chi nhánh của nhân viên tại một thời điểm | Đọc | ~100 lần/ngày | Giờ hành chính; tăng khi chốt lương | Trung bình | Điều kiện chính gồm `MaNV` và khoảng thời gian công tác |
| A-F08 | Cập nhật lương cơ bản | Điều chỉnh lương theo quyết định nhân sự | Đọc + ghi | ~1 lần/tháng/nhân viên được điều chỉnh; tối đa ~700/30 ≈ 23,3 lần/ngày nếu cả 700 người | Kỳ xét lương | Thấp | Nếu cần tính hồi cứu phải có lịch sử lương, không chỉ ghi đè lương hiện tại |
| A-F09 | Xác định chi nhánh của PT tại ngày dạy | Đối chiếu từng buổi PT với lịch sử công tác | Đọc | ~2.600 lượt đối chiếu buổi/ngày (1 buổi = 1 lượt) | Khi xem buổi dạy; dồn khối lượng khi tính hoa hồng cuối tháng | Cao | Theo giả định chung: khoảng 9.000 hội viên dùng PT, trung bình 2 buổi/tuần, tương đương khoảng 2.600 buổi diễn ra mỗi ngày. Đây là truy vấn lõi của vấn đề số 5; khi xử lý cả kỳ, hệ thống phải đối chiếu khoảng 78.000 buổi/tháng |
| A-F10 | Tính hoa hồng PT theo chi nhánh | Tổng hợp lịch hẹn hoàn thành, xác định nơi công tác tại ngày dạy và tính tiền | Đọc nặng | 1 lần/tháng/chi nhánh, tương đương 12 lần/tháng toàn hệ thống | Cuối tháng | Thấp | Phải trả kết quả đúng khi PT chuyển chi nhánh giữa tháng |
| A-F11 | Xem/xuất bảng hoa hồng | Xem chi tiết số buổi, chi nhánh và thành tiền của từng PT | Đọc | ~2 lần/tháng/chi nhánh | Cuối và đầu tháng kế tiếp | Thấp | Nên cho phép truy vết từ dòng tổng hợp về các buổi dạy nguồn |
| A-F12 | Đồng bộ vị trí hiện tại theo ngày | Cập nhật chức vụ/chi nhánh lưu hiện tại từ giai đoạn bao phủ hôm nay, không tạo thêm lịch sử | Đọc + ghi theo lô | 1 lần/ngày toàn hệ thống | 00:00 Asia/Saigon; có chạy bù khi lỗi | Thấp | QT-A04; tác vụ lặp an toàn. Thao tác dùng vị trí hiện tại đọc/đồng bộ theo lịch sử nếu tác vụ chậm; tải bổ sung đo ở GĐ2 |

## 3. Nhận xét ban đầu

1. `ChiNhanh` có rất ít dòng nhưng được tham chiếu rộng; ưu tiên tính ổn định của mã và thông tin công bố cho module khác.
2. `LichSuCongTac` là dữ liệu quan trọng nhất của Module A. Các thao tác ghi không nhiều nhưng phải bảo đảm tuyệt đối không có khoảng công tác chồng lấn.
3. Tra cứu theo nhân viên và thời điểm là mẫu truy vấn chính. Đây là cơ sở để phân tích chỉ mục ở giai đoạn vật lý, nhưng chưa chốt chỉ mục tại giai đoạn này.
4. Tính hoa hồng có tần suất thấp nhưng phải đọc nhiều lịch hẹn và thực hiện đối chiếu thời gian cho từng buổi, vì vậy có khả năng trở thành truy vấn nặng.
5. A-F09 được ước lượng khoảng 2.600 lần/ngày theo bộ giả định chung của nhóm: khoảng 9.000 hội viên có gói PT và trung bình 2 buổi/tuần. Khi tính hoa hồng cuối tháng, các lượt đối chiếu được xử lý tập trung với quy mô khoảng 78.000 buổi/tháng, đây là khoảng 78.000 lượt đối chiếu nghiệp vụ, không phải 78.000 lần gọi tác vụ. A-F09 có mức Cao theo lượt đối chiếu/ngày; A-F10 đếm lần chốt kỳ (12 lần/tháng), không cộng hai hàng thành tổng độc lập.
6. Các con số ước lượng phải được cả nhóm rà soát để nhất quán với 8–10 kịch bản sử dụng chung của hệ thống.

GD1-10: chốt hoa hồng tạo một KyHoaHong/chi nhánh/tháng và một ChiTietHoaHong/buổi hoàn thành hợp lệ; xem/xuất kỳ đã chốt đọc dữ liệu này. Chốt chạy lại không ghi thêm; cơ chế bảo đảm nhất quán thuộc GĐ2.

**GD1-15:** áp dụng [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung-gd1-15); nhãn theo lượt nghiệp vụ/ngày toàn hệ thống, độ nặng ghi riêng. Một lượt là đơn vị của chức năng ở từng hàng, không mặc định là một câu SQL hay một lần chạy tác vụ. Không cộng các chức năng bao hàm nhau hoặc lượt xử lý theo lô thành tổng giao dịch.
