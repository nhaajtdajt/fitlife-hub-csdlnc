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

# Bảng giả định số liệu chung

Đề chỉ cho 12 chi nhánh, **10.000 lượt check-in trong 17h–20h** và **> 300.000 dòng log/tháng**. Mọi module dùng **chung** bảng dưới để ước lượng tần suất (bước 1.1) và chọn index/partition (giai đoạn 2), tránh mỗi người tự đoán một số. Đây là đề xuất — cả nhóm duyệt rồi mới dùng.

## Giả định gốc

| # | Giả định | Giá trị | Cách tính / căn cứ |
|---|---|---|---|
| G1 | Lượt check-in giờ cao điểm 17h–20h (toàn hệ thống) | 10.000 | Số của đề |
| G2 | Tỷ lệ lượt vào trong 17h–20h so với cả ngày | 40% | Dữ liệu ngành: khung 3 giờ chiều tối chiếm ~28–30% ([1], [2]); Việt Nam dồn sau giờ làm nên lấy cao hơn |
| G3 | Số lần đến tập của một hội viên đang hoạt động | 2,5 lần/tuần | IHRSA: trung bình ~2 lần/tuần (104–106 lượt/năm) [3]; chỉ tính người còn đi tập nên lấy cao hơn |
| G4 | Tỷ lệ hội viên dùng PT 1-1 | 13% | IHRSA: 12–15% hội viên dùng PT [4] |
| G5 | Số buổi PT của một hội viên PT | 2 buổi/tuần | Lịch tập PT phổ biến 2–3 buổi/tuần |
| G6 | Số buổi một PT dạy mỗi ngày | 5 buổi | Thực tế PT toàn thời gian dạy 4–8 buổi/ngày [5] |
| G7 | Tỷ lệ lịch PT bị hủy | 10% | Nhóm tự đặt |
| G8 | Thời hạn gói tập trung bình | 6 tháng | Đề có gói 3/6/12 tháng; lấy giữa |
| G9 | Tỷ lệ lượt đến có mua sản phẩm | 3% | Nhóm tự đặt |
| G10 | Phân bổ hạng Silver / Gold / Platinum | 70% / 25% / 5% | Nhóm tự đặt (phụ thuộc ngưỡng ở quyết định #4) |

## Số liệu suy ra (dùng cho bảng tần suất)

| Số liệu | Giá trị | Từ | Module dùng |
|---|---|---|---|
| Lượt check-in mỗi ngày (toàn hệ thống) | ~25.000 (~2.100/chi nhánh) | G1 ÷ G2 | C, D |
| Dòng `CheckIn` mỗi tháng | ~750.000 | × 30 ngày (khớp "> 300.000" của đề) | C, D |
| Check-in giờ cao điểm | ~55 lượt/phút trung bình, đỉnh vài lượt/giây | G1 ÷ 180 phút | C |
| Hội viên đang hoạt động | ~70.000 (~5.800/chi nhánh) | 25.000 × 7 ÷ G3 | B, C |
| Tổng hội viên trong CSDL (kể cả đã hết hạn) | ~120.000 | Tự đặt, gồm hội viên cũ | B |
| Hội viên có gói PT | ~9.000 | 70.000 × G4 | B, C |
| Buổi PT diễn ra mỗi ngày | ~2.600 | 9.000 × G5 ÷ 7 | A, C |
| Lượt đặt lịch PT mỗi ngày | ~2.900 | 2.600 ÷ (1 − G7) | C |
| Lượt xem lịch trống PT mỗi ngày | ~14.500 | ~5 lần xem mỗi lượt đặt | C |
| Dòng `LichHenPT` mỗi tháng | ~87.000 | 2.900 × 30 | A, C |
| Số PT | ~520 (~45/chi nhánh) | 2.600 ÷ G6 | A, C |
| Tổng nhân viên | ~700 (~58/chi nhánh: 45 PT, 6 lễ tân, 5 bán hàng, 2 quản lý) | Tự đặt theo số PT | A |
| Điều chuyển nhân viên | ~10 lần/tháng | ~1,5%/tháng | A |
| Hóa đơn mua/gia hạn gói tập mỗi ngày | ~390 | 70.000 ÷ G8 ÷ 30 | B, D |
| Hóa đơn gói PT mỗi ngày | ~100 | Gói ~24 buổi dùng ~3 tháng | B, D |
| Hóa đơn vé Spa lẻ mỗi ngày | ~200 | Tự đặt | D |
| Hóa đơn bán sản phẩm mỗi ngày | ~750 | 25.000 × G9 | D |
| **Tổng hóa đơn mỗi ngày** | **~1.400** (~42.000/tháng) | Cộng 4 dòng trên | B, D |
| Lượt cập nhật điểm/hạng mỗi ngày | ~1.400 | Mỗi hóa đơn một lần | B |
| Số gói dịch vụ / số mã sản phẩm | ~30 / ~150 (tồn kho 150 × 12 = 1.800 dòng) | Tự đặt | B, D |

## Khối lượng sinh dữ liệu (giai đoạn 2)

Không cần sinh đủ khối lượng thật. Đề chỉ yêu cầu **≥ 70.000 dòng** cho bảng cần minh họa. Đề xuất:

| Bảng | Số dòng sinh | Ghi chú |
|---|---|---|
| `ChiNhanh` / `NhanVien` / `LichSuCongTac` | 12 / ~700 / ~1.000 | Nhỏ, đúng như thực tế |
| `HoiVien` | ~120.000 | |
| `GoiHoiVien` | ~200.000 | Mỗi hội viên vài gói theo thời gian |
| `CheckIn` | ~2.000.000 (khoảng 3 tháng) | Đủ để partition theo tháng có ý nghĩa |
| `LichHenPT` | ~260.000 (3 tháng) | |
| `HoaDon` / `ChiTietHoaDon` | ~500.000 / ~750.000 (1 năm) | 1 năm để có đủ dữ liệu xét hạng 12 tháng |

## Nguồn tham khảo

1. [What gym check-ins reveal about member behavior — Wellyx](https://wellyx.com/blog/what-gym-check-ins-reveal-about-member-behavior/)
2. [When South Africa Goes to Gym: 2.4 Million Check-Ins Analysed — Itensity](https://itensityonline.com/?p=948336)
3. [Gym membership statistics (số liệu IHRSA) — Wod.guru](https://wod.guru/blog/en/gym-membership-statistics/)
4. [IHRSA Fitness Training Report Shows Consumer Preferences — Health & Fitness Association](https://www.healthandfitness.org/ihrsa-fitness-training-report-shows-consumer-preferences/)
5. [What Does a Personal Trainer Do on a Daily Basis? — FitSW](https://www.fitsw.com/blog/what-does-a-personal-trainer-do-on-a-daily-basis/)

Các nguồn là số liệu ngành ở Mỹ/thế giới, phần lớn từ 2014–2016; chưa tìm được số liệu công khai của chuỗi gym Việt Nam. Dòng "Tự đặt" là giả định của nhóm, có thể đổi.
