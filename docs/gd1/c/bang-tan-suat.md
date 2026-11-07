# Module C — Bảng phân tích tần suất chức năng

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Chỗ đề và bảng chung không có số liệu được ghi là giả định. Quy ước thống nhất với B và D: **Cao** từ 1.000 lần/ngày; **Trung bình** 100–999 lần/ngày; **Thấp** dưới 100 lần/ngày hoặc chạy theo kỳ. Mức độ phản ánh số lần, không phản ánh độ nặng của từng truy vấn.

## Chức năng của Module C

| Mã | Chức năng | Mô tả | Loại | Tần suất toàn hệ thống | Mức độ | Căn cứ/nhận xét |
|---|---|---|---|---:|---|---|
| C-F01 | Quét vào và xác thực quyền vào | Tìm hội viên, kiểm tra phiên đang mở, gói còn hiệu lực/quyền lợi và ghi `TGVao` | Đọc + ghi | ~25.000 lần/ngày; 10.000 lượt trong 17h–20h | **Cao** | Số liệu chung; đường giao dịch phải đạt mục tiêu < 50 ms |
| C-F02 | Quét ra | Tìm lượt đang mở của hội viên và cập nhật `TGRa` | Đọc + ghi | ~25.000 lần/ngày | **Cao** | Giả định mỗi lượt vào có một lượt ra; lỗi quên quét được C-F04 xử lý |
| C-F03 | Tra cứu hội viên đang có mặt | Liệt kê các lượt chưa ra theo chi nhánh/khung giờ | Đọc | ~100 lần/ngày | Trung bình | Giả định khoảng 8 lượt/chi nhánh/ngày; D còn đọc lịch sử để báo cáo lấp đầy |
| C-F04 | Tự đóng lượt quên quét ra | Ghi `TGRa` cho các lượt còn mở tại giờ đóng cửa; `SoHieuCuaRa` giữ rỗng | Ghi theo lô | 12 lần/ngày | Thấp | Một lần/chi nhánh/ngày theo đề xuất #15; có thể tác động nhiều dòng |
| C-F05 | Quản lý cửa kiểm soát | Thêm cửa, quy định chức năng quét và kích hoạt/ngừng cửa tại chi nhánh | Ghi | ~5 lần/tháng | Thấp | Giả định; danh mục nhỏ, nhận diện bằng `(MaCN, SoHieuCua)` và được kiểm tra trên mọi lượt quét |
| C-F06 | Xem lịch trống của PT | Tra lịch chưa hủy của PT trong ngày để hiển thị các khung một giờ còn trống | Đọc | ~14.500 lần/ngày | **Cao** | Bảng giả định chung: khoảng 5 lượt xem cho mỗi lần đặt |
| C-F07 | Đặt lịch PT | Kiểm tra PT, nơi công tác, quyền lợi gói, số chỗ đã giữ và tạo lịch `Đã đặt` | Đọc + ghi | ~2.900 lần/ngày | **Cao** | Bảng giả định chung; có nguy cơ ghi đồng thời cùng PT/khung giờ |
| C-F08 | Hủy lịch PT | Chuyển lịch sang `Hủy`; hủy muộn phải trừ một buổi đúng một lần | Đọc + ghi | ~290 lần/ngày | Trung bình | Giả định 10% trong 2.900 lịch đặt |
| C-F09 | Hoàn thành/đóng lịch PT | Chuyển lịch đã diễn ra sang `Đã hoàn thành`, bảo đảm quyền lợi đã được trừ | Đọc + ghi | ~2.600 lần/ngày | **Cao** | Số buổi PT diễn ra mỗi ngày theo bảng chung |
| C-F10 | Xử lý hội viên không đến | Sau khi hết khung giờ, đóng lịch vắng mặt theo quy tắc hủy muộn và trừ buổi một lần | Đọc + ghi theo lô | ~100 lịch/ngày | Trung bình | Tự đặt: khoảng một phần ba số lịch hủy/không diễn ra |
| C-F11 | Tra cứu lịch sử tập/lịch PT | Xem check-in và lịch hẹn của một hội viên theo khoảng ngày | Đọc | ~500 lần/ngày | Trung bình | Giả định; truy vấn lịch sử tăng theo thời gian |
| C-F12 | Gửi đánh giá HLV | Hội viên chấm 1–5 điểm và nhận xét cho một lịch đã hoàn thành | Ghi | ~1.300 lần/ngày | **Cao** | Giả định 50% trong 2.600 buổi hoàn thành có đánh giá |
| C-F13 | Xem hiệu suất/đánh giá HLV | Tổng hợp số buổi hoàn thành và điểm đánh giá theo PT/kỳ | Đọc nặng | ~12 lần/ngày | Thấp | Một lần/chi nhánh/ngày; ít chạy nhưng đọc lịch sử lớn |

## Quy mô dữ liệu liên quan

- Khoảng 25.000 dòng `CheckIn` mới/ngày, 750.000 dòng/tháng; bộ dữ liệu giai đoạn 2 dự kiến khoảng 2.000.000 dòng.
- Khoảng 2.900 dòng `LichHenPT` mới/ngày, 87.000 dòng/tháng; bộ dữ liệu dự kiến khoảng 260.000 dòng.
- Tải check-in tập trung: 10.000 lượt trong ba giờ, trung bình khoảng 55 lượt/phút toàn hệ thống và có thể đạt vài lượt/giây tại đỉnh.
- Có khoảng 520 PT và 70.000 hội viên hoạt động. Danh mục cửa kiểm soát nhỏ hơn nhiều so với log.

## Nhận xét

1. `CheckIn` là bảng nóng nhất của C: vừa ghi dồn dập ở giờ cao điểm, vừa được tra cứu theo hội viên và đọc lịch sử để báo cáo. Đường quét cổng phải ngắn, không được tính báo cáo đồng bộ trong giao dịch check-in.
2. Quét vào không chỉ thêm log mà còn có thể cập nhật quyền lợi PT/Spa của B. Cờ `DaTruBuoi` và giao diện nguyên tử với B là cần thiết để yêu cầu gửi lại không trừ hai lần.
3. Xem lịch trống chạy nhiều hơn thao tác đặt thật. Mẫu lọc chính là PT + khoảng ngày + trạng thái; đây là cơ sở chọn chỉ mục ở giai đoạn 2.
4. Chống overbooking là yêu cầu toàn vẹn, không thể chỉ dựa vào bước “kiểm tra rồi thêm” ở ứng dụng. Giai đoạn 2 phải có ràng buộc tại CSDL và kiểm thử bằng hai giao dịch đồng thời.
5. C-F13 và việc tự đóng lượt có tần suất thấp nhưng đọc/cập nhật theo lô; cần đo riêng, không xếp nhẹ chỉ vì số lần chạy ít.
