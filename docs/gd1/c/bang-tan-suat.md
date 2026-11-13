# Bảng phân tích tần suất — Module C

Số liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung); chỗ bảng chung không có thì ghi **tự đặt**. Quy trình tương ứng ở [quy-trinh.md](quy-trinh.md).

**Mức độ** (toàn hệ thống, theo [quy ước chung](../../quyet-dinh.md#quy-ước-phân-loại-tần-suất-chung)): Cao ≥ 1.000 lần/ngày · Trung bình 100–999 lần/ngày · Thấp < 100 lần/ngày.

## Chức năng

| Mã | Chức năng | Mô tả | Loại | Tần suất | Mức độ | Căn cứ / nhận xét |
|---|---|---|---|---|---|---|
| C-F01 | Quét vào | Đọc gói theo `MaGHV`, kiểm tra hiệu lực và loại, trừ vé nếu có, tạo `CHECK_IN` (QT-C1) | Đọc + ghi | ~25.000 lượt/ngày; G1 minh họa ~10.000 lượt trong 17h–20h | **Cao** | G1/G2. Đề yêu cầu tra quyền vào < 50 ms, đo ở GĐ2 |
| C-F02 | Quét ra | Đóng mọi phiên mở của hội viên, ghi giờ và thiết bị ra (QT-C2) | Đọc + ghi | ≤ ~25.000 lượt/ngày | **Cao** | Mốc tối đa khi mọi phiên đều quét ra. Cập nhật dòng cũ, không tạo dòng mới |
| C-F03 | Đóng phiên quên quét ra | Job cuối ngày ghi giờ ra = giờ đóng cửa, `ThietBiRa` rỗng (QT-C3) | Ghi theo lô | 12 lần/ngày (1/chi nhánh) | Thấp | Chạy bù theo giờ đóng cửa của chính ngày đó |
| C-F04 | Tra cứu người đang có mặt | Liệt kê phiên đang mở tại chi nhánh ở thời điểm T (QT-C4) | Đọc | ~100 lần/ngày | Trung bình | Tự đặt, ~8 lượt/chi nhánh/ngày |
| C-F05 | Xem lịch trống PT | Liệt kê khung giờ tròn còn trống của các PT tại chi nhánh trong ngày (QT-C5) | Đọc | ~14.500 lần/ngày | **Cao** | Bảng chung: ~5 lượt xem mỗi lần đặt |
| C-F06 | Đặt lịch PT | Kiểm tra PT, gói PT, buổi trống, lịch trùng của hội viên; tạo lịch Đã đặt (QT-C5) | Đọc + ghi | ~2.900 lần/ngày | **Cao** | Bảng chung. Có nguy cơ hai người đặt cùng PT cùng giờ (vấn đề 2) |
| C-F07 | Hủy lịch PT | Chuyển Đã đặt → Hủy khi còn ít nhất 2 giờ (QT-C6) | Đọc + ghi | ~200 lần/ngày | Trung bình | Bảng chung (G7). Chưa tính lượt hủy bị từ chối |
| C-F08 | Tự hoàn thành và trừ buổi khi tới giờ | Job chuyển lịch Đã đặt → Đã hoàn thành và trừ 1 buổi của gói (QT-C7) | Đọc + ghi theo lô | ~2.700 lịch/ngày | **Cao** | Gồm ~100 lịch hội viên không đến. Đây cũng là thao tác trừ buổi PT của B, không cộng trùng |
| C-F09 | Tra cứu lịch sử tập và lịch PT | Hội viên/lễ tân xem check-in và lịch PT theo khoảng ngày; PT xem lịch dạy (QT-C8) | Đọc | ~500 lần/ngày | Trung bình | Tự đặt |
| C-F10 | Đánh giá HLV | Hội viên chấm 1–5 điểm cho lịch Đã hoàn thành (QT-C9) | Ghi | ~1.350 lần/ngày | **Cao** | Tự đặt: ~50% trong ~2.700 lịch hoàn thành có đánh giá |
| C-F11 | Xem hiệu suất HLV | Số buổi hoàn thành, số đánh giá, điểm trung bình theo PT (QT-C10) | Đọc nặng | ~12 lần/ngày | Thấp | Tự đặt, 1/chi nhánh/ngày. Ít lần nhưng đọc lịch sử lớn |

## Quy mô dữ liệu

- `CHECK_IN`: ~25.000 dòng mới/ngày, ~750.000 dòng/tháng; bộ dữ liệu GĐ2 dự kiến ~2.000.000 dòng.
- `LICH_HEN_PT`: ~2.900 dòng mới/ngày, ~87.000 dòng/tháng; bộ dữ liệu dự kiến ~260.000 dòng.
- ~25.000 lượt vào + tối đa ~25.000 lượt ra là ~50.000 thao tác cổng/ngày nhưng chỉ ~25.000 dòng mới. Lượt quét lặp chưa được ước lượng.
- G1 minh họa ~55,56 lượt/phút trung bình; không suy ra tải đỉnh hay mức đồng thời. Đề vẫn yêu cầu >10.000 lượt quét đồng thời ([phân biệt tải cổng](../../quyet-dinh.md#phân-biệt-tải-cổng-và-giả-định)).

## Nhận xét

1. `CHECK_IN` là bảng nóng nhất: ghi dồn dập giờ cao điểm, lại là nguồn cho báo cáo lấp đầy. Đường quét vào chỉ đọc một gói theo khóa chính và ghi một dòng; báo cáo không chạy trong giao dịch này.
2. Quét ra tìm phiên mở qua các gói của hội viên (`CHECK_IN` không có `MaHV`), nên cần chỉ mục phù hợp ở GĐ2.
3. Xem lịch trống chạy nhiều gấp ~5 lần đặt thật. Mẫu lọc chính là PT + ngày + trạng thái.
4. Chống trùng lịch PT không thể chỉ dựa vào bước "kiểm tra rồi thêm" ở ứng dụng; phải có ràng buộc duy nhất tại CSDL và kiểm thử bằng hai giao dịch đồng thời ở GĐ2.
5. C-F08 chạy theo lô mỗi giờ nhưng mỗi lịch là một giao dịch nhỏ; lịch đã chuyển trạng thái thì không bị trừ lại.
