# Module C — Bảng phân tích tần suất chức năng

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Chỗ đề và bảng chung không có số liệu được ghi là giả định. Quy ước thống nhất toàn A/B/C/D: **Cao** từ 1.000 lần/ngày; **Trung bình** 100–999 lần/ngày; **Thấp** dưới 100 lần/ngày sau quy đổi. Mức độ phản ánh số lần, không phản ánh độ nặng của từng truy vấn.

## Chức năng của Module C

| Mã | Chức năng | Mô tả | Loại | Tần suất toàn hệ thống | Mức độ | Căn cứ/nhận xét |
|---|---|---|---|---:|---|---|
| C-F01 | Quét vào và xác thực quyền vào | Tìm hội viên, kiểm tra phiên đang mở, gói còn hiệu lực/quyền lợi và ghi `TGVao` | Đọc + ghi | ~25.000 lượt vào thành công/ngày; G1 minh họa ~10.000 lượt cộng dồn trong 17h–20h | **Cao** | Giả định G1/G2; đề yêu cầu tra cứu quyền vào < 50 ms, chưa đo/kiểm chứng; không thay yêu cầu đồng thời bằng trung bình |
| C-F02 | Quét ra | Tìm lượt đang mở của hội viên và cập nhật `TGRa` | Đọc + ghi | ~25.000 lần/ngày | **Cao** | Mốc tối đa khi mọi phiên đều quét ra; quên quét được C-F04 đóng thay. Cập nhật dòng cũ, không tạo thêm CheckIn |
| C-F03 | Tra cứu hội viên đang có mặt | Liệt kê các lượt chưa ra theo chi nhánh/khung giờ | Đọc | ~100 lần/ngày | Trung bình | Giả định khoảng 8 lượt/chi nhánh/ngày; D còn đọc lịch sử để báo cáo lấp đầy |
| C-F04 | Tự đóng lượt quên quét ra | Ghi `TGRa` cho các lượt còn mở tại giờ đóng cửa; `SoHieuCuaRa` giữ rỗng | Ghi theo lô | 12 lần/ngày | Thấp | Một lần/chi nhánh/ngày theo quy ước mô phỏng #15; chạy bù khi lỗi, không ghi đè lượt đã đóng |
| C-F05 | Quản lý cửa kiểm soát | Thêm cửa, quy định chức năng quét và kích hoạt/ngừng cửa tại chi nhánh | Ghi | ~5 lần/tháng | Thấp | Giả định; danh mục nhỏ, nhận diện bằng `(MaCN, SoHieuCua)` và được kiểm tra trên mọi lượt quét |
| C-F06 | Xem lịch trống của PT | Tra lịch chưa hủy của PT trong ngày để hiển thị các khung một giờ còn trống | Đọc | ~14.500 lần/ngày | **Cao** | Bảng giả định chung: khoảng 5 lượt xem cho mỗi lần đặt |
| C-F07 | Đặt lịch PT | Kiểm tra PT, nơi công tác, quyền lợi gói, số chỗ đã giữ và tạo lịch `Đã đặt` | Đọc + ghi | ~2.900 lần/ngày | **Cao** | Bảng giả định chung; có nguy cơ ghi đồng thời cùng PT/khung giờ |
| C-F08 | Hủy lịch PT | Chấp nhận hủy khi còn ít nhất 2 giờ, giải phóng buổi giữ; từ chối hủy muộn | Đọc + ghi | ~200 lần hủy thành công/ngày | Trung bình | Giả định cập nhật: 2.900 lịch = 2.600 hoàn thành + 100 vắng + 200 hủy đúng hạn; chưa tính yêu cầu hủy bị từ chối |
| C-F09 | Hoàn thành/đóng lịch PT | Chuyển lịch đã diễn ra sang `Đã hoàn thành`, bảo đảm quyền lợi đã được trừ | Đọc + ghi | ~2.600 lần/ngày | **Cao** | Số buổi PT diễn ra mỗi ngày theo bảng chung |
| C-F10 | Xử lý hội viên không đến | Sau khi hết khung giờ, xác nhận vắng mặt; buổi đã trừ tại giờ hẹn, không trừ thêm | Đọc + ghi theo lô | ~100 lượt xác nhận vắng/ngày (1 lịch = 1 lượt) | Trung bình | Tự đặt: 100 trong 300 lịch không hoàn thành; tách riêng 200 hủy đúng hạn |
| C-F11 | Tra cứu lịch sử tập/lịch PT | Xem check-in và lịch hẹn của một hội viên theo khoảng ngày | Đọc | ~500 lần/ngày | Trung bình | Giả định; truy vấn lịch sử tăng theo thời gian |
| C-F12 | Gửi đánh giá HLV | Hội viên chấm 1–5 điểm và nhận xét cho một lịch đã hoàn thành | Ghi | ~1.300 lần/ngày | **Cao** | Giả định 50% trong 2.600 buổi hoàn thành có đánh giá |
| C-F13 | Xem hiệu suất/đánh giá HLV | Tổng hợp số buổi hoàn thành và điểm đánh giá theo PT/kỳ | Đọc nặng | ~12 lần/ngày | Thấp | Một lần/chi nhánh/ngày; ít chạy nhưng đọc lịch sử lớn |
| C-F14 | Tự trừ buổi PT đến giờ hẹn | Quét lịch đến hạn chưa hủy/chưa trừ, tiêu thụ buổi đã giữ và đặt cờ nguyên tử | Đọc + ghi theo lô | ~2.700 lượt trừ lịch/ngày (1 lịch = 1 lượt) | Cao | 2.600 hoàn thành + 100 vắng mặt; không cộng thêm lần trừ khi đóng lịch |

## Quy mô dữ liệu liên quan

- Khoảng 25.000 dòng `CheckIn` mới/ngày, 750.000 dòng/tháng; bộ dữ liệu giai đoạn 2 dự kiến khoảng 2.000.000 dòng.
- Khoảng 2.900 dòng `LichHenPT` mới/ngày, 87.000 dòng/tháng; bộ dữ liệu dự kiến khoảng 260.000 dòng.
- G1 minh họa ~10.000 lượt cộng dồn/3 giờ → ~55,56 lượt/phút = ~0,926 lượt/giây trung bình. Không suy ra tải đỉnh hoặc mức đồng thời. Đề vẫn yêu cầu >10.000 lượt quét đồng thời toàn hệ thống; xem cách phân biệt và giới hạn giả định ở GD1-16 trong bảng chung.
- ~25.000 vào + tối đa ~25.000 quét ra/ngày là ~50.000 thao tác cổng, chỉ ~25.000 dòng mới; tự đóng thay cho lượt quên ra. Lượt quét lỗi/lặp chưa được tính; 230 lượt dùng vé (gồm mua lại) đã nằm trong lượt vào, không cộng trùng.
- Có khoảng 520 PT và 70.000 hội viên hoạt động. Danh mục cửa kiểm soát nhỏ hơn nhiều so với log.

## Nhận xét

1. `CheckIn` là bảng nóng nhất của C: vừa ghi dồn dập ở giờ cao điểm, vừa được tra cứu theo hội viên và đọc lịch sử để báo cáo. Đường quét cổng phải ngắn, không được tính báo cáo đồng bộ trong giao dịch check-in.
2. Quét vào chỉ kiểm tra thẻ/vé vào và có thể trừ vé, không đọc/trừ PT. Tác vụ đến giờ hẹn xử lý PT riêng; `DaTruBuoi` và giao dịch với B ngăn trừ lặp khi tác vụ chạy lại.
3. Xem lịch trống chạy nhiều hơn thao tác đặt thật. Mẫu lọc chính là PT + khoảng ngày + trạng thái; đây là cơ sở chọn chỉ mục ở giai đoạn 2.
4. Chống overbooking là yêu cầu toàn vẹn, không thể chỉ dựa vào bước “kiểm tra rồi thêm” ở ứng dụng. Giai đoạn 2 phải có ràng buộc tại CSDL và kiểm thử bằng hai giao dịch đồng thời.
5. C-F13 và việc tự đóng lượt có tần suất thấp nhưng đọc/cập nhật theo lô; cần đo riêng, không xếp nhẹ chỉ vì số lần chạy ít.

Các số PT trên là giả định cập nhật theo quy tắc mới, đã đối chiếu cơ cấu ở GD1-15; tải/đồng thời đã tách đơn vị ở GD1-16; số lần chạy tác vụ khác số lịch xử lý. Vé mới dùng giả định G11–G12, không làm tăng tổng vào thêm lần nữa.

**GD1-06:** mỗi lần đặt lịch phải xét lại lịch sử công tác gồm giai đoạn chờ; đổi vị trí ở A phải xét lịch chưa hủy bị ảnh hưởng. Đây là kiểm tra bên trong nghiệp vụ hiện có, không cộng thành lượt đặt lịch mới.

**GD1-15:** áp dụng [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung-gd1-15); nhãn theo lượt nghiệp vụ/ngày toàn hệ thống, độ nặng ghi riêng. Một lượt là đơn vị của chức năng ở từng hàng, không mặc định là một câu SQL hay một lần chạy tác vụ. Không cộng các chức năng bao hàm nhau hoặc lượt xử lý theo lô thành tổng giao dịch.

**Xử lý theo đợt:** trừ PT tại giờ hẹn; không chờ cuối giờ/cuối ngày. Cách gom lịch, kích hoạt tác vụ và chạy bù thuộc [ghi chú GĐ2](../../gd2/ghi-chu-trien-khai.md). CN-B08 và C-F14 là cùng một nghiệp vụ, không cộng trùng.
