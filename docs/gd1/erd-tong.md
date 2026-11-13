# ERD mức quan niệm — đang vẽ lại

ERD đang được vẽ lại cho khớp [lược đồ logic chốt](luoc-do-logic.png) và [quyết định của nhóm](../quyet-dinh.md). [conceptual-diagram.png](conceptual-diagram.png) là bản ERD trước khi chốt (10/11/2026), chỉ dùng làm nền tham khảo.

## Những điểm ERD mới cần thể hiện

| Thực thể / mối kết hợp | Ghi chú khi vẽ |
|---|---|
| CHI NHÁNH | Không có thực thể tiện ích riêng (quyết định #20) |
| NHÂN VIÊN | Phân cấp PT / Lễ tân / Quản lý chi nhánh / Nhân viên bán hàng, ánh xạ thành cột `ChucVu` ở mức logic |
| CÔNG TÁC (lịch sử) | NHÂN VIÊN 1–n, CHI NHÁNH 1–n; ngày bắt đầu/kết thúc |
| LỊCH SỬ HOA HỒNG | CHI NHÁNH 1–n; mức mỗi buổi theo khoảng ngày |
| HỘI VIÊN | Đăng ký tại một CHI NHÁNH; có tổng chi tiêu, điểm, hạng |
| HẠNG MỤC CUNG CẤP | Gộp gói và sản phẩm (quyết định #3); phân loại Gym / Yoga / PT / Spa / Vé lượt / Sản phẩm |
| GÓI HỘI VIÊN (lần mua) | HỘI VIÊN 1–n, HẠNG MỤC 1–n; ngày kích hoạt, hết hạn, số buổi còn lại |
| CHECK-IN | GÓI HỘI VIÊN 1–n (quét bằng mã gói, quyết định #8), CHI NHÁNH 1–n; thiết bị vào/ra là thuộc tính, không có thực thể cửa kiểm soát (#19) |
| LỊCH HẸN PT | GÓI HỘI VIÊN (gói PT) 1–n, PT 1–n, CHI NHÁNH 1–n |
| ĐÁNH GIÁ | LỊCH HẸN 1–0..1 |
| TỒN KHO | Mối kết hợp n–n CHI NHÁNH – HẠNG MỤC (loại Sản phẩm), thuộc tính số lượng tồn |
| HÓA ĐƠN | HỘI VIÊN 1–n, NHÂN VIÊN lập 1–n, CHI NHÁNH 1–n |
| CHI TIẾT HÓA ĐƠN | Mối kết hợp n–n HÓA ĐƠN – HẠNG MỤC, thuộc tính số lượng, đơn giá |

Ràng buộc không thể hiện được trên hình: xem mục 6 của [thiết kế logic tổng](thiet-ke-tong.md) (R01–R27).
