# Quyết định làm rõ yêu cầu

Những chỗ đề chưa rõ. Cột "Đề xuất" là **chưa chốt** — cả nhóm chốt ở Sprint 0, ghi kết quả vào cột "Chốt". Chỗ nào không chắc thì hỏi cô.

| # | Câu hỏi | Đề xuất | Module bị ảnh hưởng | Chốt |
|---|---|---|---|---|
| 1 | Gói tập dùng được ở chi nhánh nào? | Mọi chi nhánh (đề: "bất kỳ chi nhánh nào") | C | |
| 2 | Gói PT 1-1 chọn HLV thế nào? | Chọn HLV từng lần đặt lịch; mỗi buổi hoàn thành trừ 1 buổi còn lại của gói | B, C | |
| 3 | Một hóa đơn gồm gì? | Cả gói và sản phẩm, qua `ChiTietHoaDon` | B, D | |
| 4 | Hạng tính theo "năm" hay "12 tháng qua"? (đề tự mâu thuẫn) | 12 tháng gần nhất; ngưỡng nhóm tự đặt, ví dụ Gold ≥ 20 triệu, Platinum ≥ 50 triệu | B | |
| 5 | Điểm Loyalty tích trên gì? | Tổng tiền sau giảm giá, 1 điểm / 100.000đ; chưa làm đổi điểm | B, D | |
| 6 | Hoa hồng PT tính thế nào? | Mức cố định mỗi buổi hoàn thành, theo chi nhánh PT thuộc về **tại ngày buổi tập** | A, C | |
| 7 | Điểm đánh giá HLV, sức chứa phòng tập (đề có nhắc ở truy vấn nhưng không mô tả) | Bổ sung `DanhGiaHLV` và cột sức chứa cho chi nhánh | A, C, D | |
