# Hồ sơ tổng giai đoạn 1 — FitLife Hub

Bản ghép A/B/C/D sau rà soát 17 mục. **Rà cuối phát hiện 8 nhóm việc còn cần xử lý; xem [báo cáo rà soát cuối](ra-soat-cuoi.md). Chưa coi đây là bản cuối sẵn sàng nộp.** Chỉ bao gồm nghiệp vụ, tần suất, mô hình quan niệm và thiết kế logic theo [yêu cầu GĐ1](../../required.md).

| Đầu ra yêu cầu | Tài liệu |
|---|---|
| Quy trình nghiệp vụ chi tiết | [A](a/quy-trinh.md), [B](b/quy-trinh.md), [C](c/quy-trinh.md), [D](d/quy-trinh.md) |
| Chức năng, mô tả, tần suất và mức độ | [A](a/bang-tan-suat.md), [B](b/bang-tan-suat.md), [C](c/bang-tan-suat.md), [D](d/bang-tan-suat.md); 52 hàng dùng [giả định chung](../quyet-dinh.md) |
| ERD tổng và ràng buộc ngoài hình | [ERD tổng](erd-tong.md), [ràng buộc tổng](thiet-ke-tong.md) |
| 20 lược đồ, khóa, PTH, dạng chuẩn, 33 FK | [Thiết kế tổng](thiet-ke-tong.md) |
| Thông tin quan hệ/thuộc tính | [A](a/thiet-ke-logic.md), [B](b/thiet-ke-logic.md), [C](c/thiet-ke-logic.md), [D](d/thiet-ke-logic.md), dẫn chiếu từ bản tổng |
| Bối cảnh, nội dung, tầm ảnh hưởng ràng buộc | Bảng T-R01–14 trong [thiết kế tổng](thiet-ke-tong.md), cùng các bảng nội module A/B/C/D |
| Quyết định, giả định, giới hạn và dấu vết sửa lỗi | [Quyết định chung](../quyet-dinh.md), [sổ rà soát](theo-doi-ra-soat.md) |

Dùng ERD tổng và các bản erd.md hiện hành; sơ đồ A drawio/C PNG cũ không đủ các cập nhật tích hợp. Không dùng bản vẽ cũ thay cho bản tổng khi nộp. Bản tổng thống nhất bảng cơ sở với quan hệ dẫn xuất vGoiHoiVien; không tạo bảng trùng theo module.

Kết luận chuẩn hóa: 19 quan hệ BCNF và HoiVien 2NF do giữ Hang phụ thuộc TongChiTieu12T; toàn lược đồ không được kết luận đạt 3NF/BCNF. Lịch PT đã chốt khung một giờ bắt đầu vào giờ tròn; trừ tại giờ hẹn, không phụ thuộc cổng.

Trước khi nộp: bổ sung tên/MSSV còn thiếu và xác nhận ngưỡng hạng với giảng viên. Các giả định/ngoài phạm vi khác được công bố trong thiết kế tổng. Ghi chú triển khai nằm riêng ở GĐ2; không phải phần đã cài đặt hoặc kiểm thử.
