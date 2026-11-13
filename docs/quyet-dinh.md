# Quyết định làm rõ yêu cầu

Những chỗ đề chưa rõ và quyết định của nhóm. Bản này đã cập nhật theo [lược đồ logic chốt](gd1/luoc-do-logic.png) (13 bảng) ngày 13/11/2026; các quyết định cũ không còn phù hợp đã được thay, không giữ song song.

## Quyết định nghiệp vụ

| # | Câu hỏi | Chốt | Module |
|---|---|---|---|
| 1 | Gói tập dùng được ở chi nhánh nào? | Mọi chi nhánh (đề: "bất kỳ chi nhánh nào") | B, C |
| 2 | Gói PT trừ buổi khi nào? | Đến giờ tập, job chuyển lịch **Đã đặt** sang **Đã hoàn thành** và trừ 1 buổi của đúng gói đã gắn với lịch, trong cùng giao dịch. Lịch đã chuyển trạng thái thì không bị trừ lần hai, nên không cần cờ "đã trừ buổi". Không trừ khi check-in | B, C |
| 3 | Hóa đơn gồm gì? Gói và sản phẩm lưu thế nào? | Một hóa đơn gồm cả gói và sản phẩm qua `CHI_TIET_HOA_DON`. Gói và sản phẩm **gộp chung** bảng `HANG_MUC_CUNG_CAP`, phân biệt bằng `LoaiHangMuc` ∈ {Gym, Yoga, PT, Spa, Vé lượt, Sản phẩm} | B, D |
| 4 | Hạng tính theo "năm" hay "12 tháng qua"? (đề tự mâu thuẫn) | Theo **năm dương lịch**, đúng chữ "tổng chi tiêu trong năm". Thanh toán thì cộng dồn `TongChiTieu`. 00:00 ngày 1/1 job tính lại `TongChiTieu` từ hóa đơn của năm mới (bình thường về 0) và hạng về Silver. Ngưỡng: Silver < 20tr ≤ Gold < 50tr ≤ Platinum, do nhóm quyết định | B |
| 5 | Điểm Loyalty tích trên gì? | Phần nguyên `TongTien / 100.000` của mỗi hóa đơn (sau giảm giá). Điểm không hết hạn, không reset đầu năm, chưa hỗ trợ đổi quà | B, D |
| 6 | Hoa hồng PT tính thế nào? | Mỗi buổi **Đã hoàn thành** tính một lần, cho chi nhánh PT thuộc về tại `NgayTap`, với mức của chi nhánh đó tại `NgayTap` trong `HOA_HONG_CHI_NHANH` | A, C |
| 7 | Điểm đánh giá HLV, sức chứa phòng tập (đề nhắc ở truy vấn, không mô tả) | Thêm bảng `DANH_GIA_HLV` (một lịch tối đa một đánh giá, điểm 1–5) và cột `CHI_NHANH.SucChua` | A, C, D |
| 8 | Quyền vào cổng và điều kiện mua PT? | Hội viên quét **mã QR của từng gói** (`MaGHV`). Vào được khi gói còn hiệu lực theo ngày, còn lượt (nếu có giới hạn) và **không phải PT**. Vé Spa/vé lượt bị trừ 1 lượt cùng giao dịch check-in. Mua PT **không** bắt buộc có thẻ Gym; muốn vào tập thì dùng thẻ hoặc mua vé | B, C, D |
| 9 | Gói bắt đầu khi nào? Hạn bao lâu? Bảo lưu? | Mọi gói **bắt đầu ngày mua**, không gia hạn nối tiếp; mua sớm thì phần chồng không được bù. Gym/Yoga/PT: hết hạn = ngày mua + số tháng − 1 ngày (tháng đích thiếu ngày thì lấy cuối tháng). Spa và vé lượt: hạn 7 ngày (ngày mua + 6), 1 lượt. Bảo lưu ngoài phạm vi | B |
| 10 | Khung giờ PT? | Một giờ, bắt đầu vào giờ tròn, nằm trọn trong giờ mở cửa chi nhánh. Trong các lịch chưa hủy, `(MaNV, NgayTap, ThoiGianBatDau)` là duy nhất | C |
| 11 | Hủy lịch PT trước bao lâu? Không đến thì sao? | Chỉ hủy khi còn ít nhất 2 giờ; hủy không trừ buổi. Không hủy mà không đến thì vẫn là **Đã hoàn thành**: trừ buổi, PT vẫn được tính hoa hồng (khung giờ đã bị giữ), hội viên vẫn được đánh giá | B, C, A |
| 12 | PT có được dạy ở chi nhánh khác không? | Không, chỉ dạy tại chi nhánh đang công tác và phải có chức vụ PT tại ngày tập. Đổi chi nhánh/chức vụ/nghỉ việc bị từ chối nếu còn lịch Đã đặt từ ngày hiệu lực trở đi không còn hợp lệ; hệ thống không tự hủy hay đổi lịch | A, C |
| 13 | Ngày công tác và chức vụ lưu thế nào? | DATE, tính cả hai biên. Thay đổi gửi trong ngày D: giai đoạn cũ kết thúc D, mới bắt đầu D+1. `NHAN_VIEN.MaCNHienTai` và `ChucVu` lưu sẵn, job đầu ngày đồng bộ từ lịch sử. Không có giai đoạn bao phủ hôm nay thì `MaCNHienTai` rỗng = **đã nghỉ việc** | A |
| 14 | Sửa mức hoa hồng thì các tháng cũ có đổi không? | Không. Mức lưu theo khoảng ngày trong `HOA_HONG_CHI_NHANH`, mức mới chỉ có hiệu lực từ ngày mai. Lịch PT của tháng đã qua không được sửa. Vì vậy hoa hồng tính lại lúc nào cũng ra cùng kết quả, **không lưu kết quả chốt kỳ** | A, C |
| 15 | Tỷ lệ lấp đầy tính thế nào? Quên quét ra thì sao? | Chỉ báo cáo, không chặn cổng. Tại T: số `CHECK_IN` cùng chi nhánh có `ThoiGianVao <= T` và (`ThoiGianRa` rỗng hoặc `> T`), chia `SucChua`, có thể vượt 100%. Phiên quên quét ra được job đóng tại giờ đóng cửa, `ThietBiRa` rỗng | C, D |
| 16 | Hủy/hoàn tiền hóa đơn? Hạ hạng khi nào? | Hóa đơn đã thanh toán không sửa, không xóa, không hoàn tiền. Trong năm hạng chỉ tăng; chỉ về Silver khi reset đầu năm (#4) | B, D |
| 17 | Web hay WinForm, ngôn ngữ, phiên bản SQL Server? | Chưa chốt (GĐ2) | Cả nhóm |
| 18 | Doanh thu trước hay sau giảm giá? | Doanh thu thực thu (tổng, theo chi nhánh/thời gian) dùng `TongTien`, **sau giảm giá**. Doanh thu theo từng hàng mục dùng `SoLuong × DonGia`, **trước giảm giá**, ghi rõ trên báo cáo; không phân bổ giảm giá xuống dòng | D |
| 19 | Cửa kiểm soát có cần bảng riêng? | Không. Thiết bị quét ghi dạng chữ trong `CHECK_IN.ThietBiVao` / `ThietBiRa` | C |
| 20 | "Danh sách dịch vụ/tiện ích tại chi nhánh" lưu ở đâu? | Dịch vụ là các hàng mục gói, có ở mọi chi nhánh. Sản phẩm bán ở chi nhánh nào thể hiện bằng dòng `TON_KHO(MaCN, MaHM)`; hết hàng thì tồn = 0 và giữ dòng; ngừng bán tại chi nhánh thì xóa dòng khi tồn = 0. Không có bảng tiện ích riêng | A, D |
| 21 | Chi nhánh có ngừng hoạt động không? | Không. Cố định 12 chi nhánh, chỉ thêm và sửa | A |
| 22 | Check-in vào/ra xử lý thế nào? | **Vào:** kiểm tra gói, trừ vé nếu có, tạo phiên; không kiểm tra phiên đang mở. **Ra:** đóng mọi phiên mở của hội viên; phiên mới nhất ghi thiết bị thật, phiên cũ hơn để `ThietBiRa` rỗng. **Cuối ngày:** đóng phiên còn sót tại giờ đóng cửa. Chống dùng chung mã QR và phân khu Gym/Yoga ngoài phạm vi | C |
| 23 | Ai quản lý `HANG_MUC_CUNG_CAP`? | B quản lý các dòng loại gói (Gym, Yoga, PT, Spa, Vé lượt); D quản lý loại Sản phẩm và tồn kho | B, D |
| 24 | Hóa đơn thuộc chi nhánh nào? Hội viên thuộc chi nhánh nào? | Hóa đơn: chi nhánh hiện tại của nhân viên lập. Hội viên: chi nhánh nơi đăng ký (`HOI_VIEN.MaCN`), dùng cho danh sách cấp chi nhánh | B, D |
| 25 | Một hội viên có được đặt hai PT cùng giờ? | Không. Một hội viên không có hai lịch chưa hủy trùng ngày, trùng giờ bắt đầu | C |
| 26 | Có lưu nhật ký nhập/xuất kho không? | Không. Chỉ lưu số tồn hiện tại; lượng đã bán tra từ `CHI_TIET_HOA_DON` | D |
| 27 | Trạng thái lịch PT lưu thế nào? | `TrangThai` không rỗng: 0 = Đã đặt (mặc định), 1 = Hủy, 2 = Đã hoàn thành. Chỉ có 0 → 1 (hủy đúng hạn) và 0 → 2 (job tới giờ) | C |
| 28 | Kết luận dạng chuẩn? | `HOI_VIEN` (`TongChiTieu → Hang`), `LICH_HEN_PT` (`{MaNV, NgayTap} → MaCN`) và `HOA_DON` (`{MaNV, NgayLap} → MaCN`) không đạt 3NF, đều **cố ý giữ** để truy vấn nhanh. Bỏ `ThoiGianKetThuc` khỏi `LICH_HEN_PT`. 10 quan hệ còn lại đạt BCNF. **Toàn lược đồ đạt 2NF**. Chi tiết tại [thiết kế logic](gd1/thiet-ke-tong.md) | Cả nhóm |

## Quy ước phân loại tần suất chung

- Thấp: dưới 100 lần/ngày; Trung bình: từ 100 đến dưới 1.000 lần/ngày; Cao: từ 1.000 lần/ngày. Áp dụng toàn hệ thống, không so tổng một module với số riêng một chi nhánh.
- Với lịch tuần/tháng: quy đổi trung bình theo 7 ngày/tuần, 30 ngày/tháng, nhân số chi nhánh/nhân viên được áp dụng; vẫn ghi số gốc và thời điểm chạy dồn. Không tự gọi mọi tác vụ theo kỳ là thấp.
- Tần suất GĐ1 đếm lượt nghiệp vụ theo đơn vị ghi ở từng hàng (một lượt quét, một lịch, một hóa đơn…). Không đồng nhất lượt nghiệp vụ với số dòng đọc, số câu SQL hay số lần chạy job. Chi phí kỹ thuật đo riêng ở GĐ2.
- Bảng tần suất là danh sách chức năng, không phải các giao dịch rời nhau. Không cộng tất cả hàng thành tổng: một lần thanh toán gồm D lập hóa đơn và B tạo gói, cộng điểm; trừ vé nằm trong check-in; trừ buổi PT nằm trong job tới giờ của C.
- Bộ giả định PT: ~2.900 lịch/ngày = ~2.700 lịch Đã hoàn thành (gồm ~100 lịch hội viên không đến) + ~200 lịch hủy đúng hạn. ~2.700 lịch bị trừ buổi và được tính hoa hồng. Các con số là giả định, không phải số đo.

# Bảng giả định số liệu chung

Đề cho 12 chi nhánh và ghi “> 10.000 lượt quét thẻ/QR check-in đồng thời toàn hệ thống” trong 17h–20h; log check-in và lịch sử tập luyện tăng > 300.000 dòng/tháng (de.md, vấn đề 1 và 3). Không thay câu này bằng 10.000 lượt cộng dồn trong ba giờ. Bảng dưới là bộ giả định làm việc của GĐ1, không phải số đo và không chứng minh đáp ứng yêu cầu tải của đề. Cả A/B/C/D dùng chung; cơ chế và phép đo thuộc GĐ2.

## Giả định gốc

| # | Giả định | Giá trị | Cách tính / căn cứ |
|---|---|---|---|
| G1 | Mốc minh họa số lượt vào thành công cộng dồn trong 17h–20h | ~10.000 lượt/3 giờ toàn hệ thống | Giả định làm tròn dùng tính tần suất; không phải diễn giải hay đáp ứng yêu cầu “> 10.000 ... đồng thời” của đề |
| G2 | Tỷ lệ lượt vào trong 17h–20h so với cả ngày | 40% | Giả định nhóm để tính tần suất |
| G3 | Số lần đến tập của một hội viên đang hoạt động | 2,5 lần/tuần | IHRSA: trung bình ~2 lần/tuần (104–106 lượt/năm) [3]; chỉ tính người còn đi tập nên lấy cao hơn |
| G4 | Tỷ lệ hội viên dùng PT 1-1 | 13% | IHRSA: 12–15% hội viên dùng PT [4] |
| G5 | Số buổi PT của một hội viên PT | 2 buổi/tuần | Lịch tập PT phổ biến 2–3 buổi/tuần |
| G6 | Số buổi một PT dạy mỗi ngày | 5 buổi | Thực tế PT toàn thời gian dạy 4–8 buổi/ngày [5] |
| G7 | Tỷ lệ lịch PT hủy đúng hạn | ~7% | Trong ~2.900 lịch/ngày, ~200 hủy đúng hạn; ~100 lịch không đến vẫn tính Đã hoàn thành (#11) |
| G9 | Tỷ lệ lượt đến có mua sản phẩm | 3% | Nhóm tự đặt |
| G10 | Phân bổ hạng Silver / Gold / Platinum | 70% / 25% / 5% | Nhóm tự đặt (phụ thuộc ngưỡng ở quyết định #4) |
| G11 | Cơ cấu ~390 lượt mua Gym/Yoga/ngày | 300 gói 3/6/12 tháng + 60 gói 1 tháng + 30 vé lượt | Gồm mua mới và mua tiếp; trong 30 vé có 10 lượt mua lại khi quay vào sau khi đã ra |
| G12 | Lượt dùng vé tại cổng/ngày | 230 = 200 Spa + 30 vé lượt Gym/Yoga | Giả định vé được dùng ngay trong ngày mua (dù hạn 7 ngày); đã nằm trong tổng 25.000 lượt vào, không cộng lần nữa |

## Số liệu suy ra (dùng cho bảng tần suất)

| Số liệu | Giá trị | Từ | Module dùng |
|---|---|---|---|
| Lượt check-in mỗi ngày (toàn hệ thống) | ~25.000 (~2.100/chi nhánh) | G1 ÷ G2 | C, D |
| Dòng `CHECK_IN` mỗi tháng | ~750.000 | 25.000 phiên × 30 ngày; một phiên một dòng, quét ra cập nhật dòng cũ | C, D |
| Trung bình của mốc minh họa G1 | ~55,56 lượt/phút = ~0,926 lượt/giây toàn hệ thống | 10.000 ÷ 180 phút; không suy ra đỉnh/đồng thời | C |
| Hội viên đang hoạt động | ~70.000 (~5.800/chi nhánh) | 25.000 × 7 ÷ G3 | B, C |
| Tổng hội viên trong CSDL (kể cả đã hết hạn) | ~120.000 | Tự đặt, gồm hội viên cũ | B |
| Hội viên có gói PT | ~9.000 | 70.000 × G4 | B, C |
| Lịch PT Đã hoàn thành mỗi ngày | ~2.700 (~2.600 có tập + ~100 không đến) | 9.000 × G5 ÷ 7, cộng lịch không đến | A, C |
| Lượt đặt lịch PT mỗi ngày | ~2.900 | 2.700 + 200 hủy đúng hạn | C |
| Lượt xem lịch trống PT mỗi ngày | ~14.500 | ~5 lần xem mỗi lượt đặt | C |
| Dòng `LICH_HEN_PT` mỗi tháng | ~87.000 | 2.900 × 30 | A, C |
| Số PT | ~520 (~45/chi nhánh) | 2.600 ÷ G6 | A, C |
| Tổng nhân viên | ~700 (~58/chi nhánh: 45 PT, 6 lễ tân, 5 bán hàng, 2 quản lý) | Tự đặt theo số PT | A |
| Điều chuyển nhân viên | ~10 lần/tháng | ~1,5%/tháng | A |
| Lượt mua Gym/Yoga mỗi ngày | ~390 gói/vé | G11 | B, D |
| Hóa đơn gói PT mỗi ngày | ~100 | Gói ~24 buổi dùng ~3 tháng | B, D |
| Hóa đơn vé Spa mỗi ngày | ~200 | Tự đặt | D |
| Hóa đơn bán sản phẩm mỗi ngày | ~750 | 25.000 × G9 | D |
| **Tổng hóa đơn mỗi ngày** | **~1.400** (~42.000/tháng) | 690 hóa đơn có gói + 750 có sản phẩm − 40 có cả hai | B, D |
| Lượt cập nhật điểm/hạng mỗi ngày | ~1.400 | Mỗi hóa đơn một lần | B |
| Số hàng mục gói / sản phẩm | ~30 / ~150 (tồn kho tối đa 150 × 12 = 1.800 dòng) | Tự đặt | B, D |

## Phân biệt tải cổng và giả định

- **Lượt trong khoảng:** G1 là mốc minh họa cộng dồn trong ba giờ; G1/G2 cho ~25.000 lượt vào thành công/ngày. Một người có thể vào nhiều lần, nên đây không phải 25.000 người khác nhau.
- **Tốc độ trung bình:** ~55,56 lượt/phút chỉ đúng với mốc minh họa G1, không khẳng định các chi nhánh có tải bằng nhau.
- **Tốc độ đỉnh:** số lượt trong giây/phút bận nhất; đề chưa định lượng và không suy được từ trung bình.
- **Mức đồng thời:** số yêu cầu đang được xử lý chồng thời gian, khác lượt/giây và khác số người trong phòng. Giữ nguyên yêu cầu >10.000 của đề; cách tạo tải cần làm rõ trước kiểm thử GĐ2.
- **Số dòng:** mỗi lượt vào thành công tạo một `CHECK_IN`; quét ra cập nhật dòng đã có. ~25.000 vào + tối đa ~25.000 quét ra là ~50.000 thao tác cổng/ngày, không phải 50.000 dòng mới. Quét vào lặp tạo thêm phiên (#22); số lượt lặp chưa có ước lượng.
- **Mục tiêu <50 ms:** mục tiêu của đề, chưa phải kết quả; phải phân biệt thời gian truy vấn với toàn bộ lượt quét khi đo ở GĐ2.
- **Gói/vé mới:** tổng tạo gói/vé ~690/ngày = 390 Gym/Yoga + 100 PT + 200 Spa; trừ vé tại cổng ~230/ngày là phần con của check-in.

## Khối lượng sinh dữ liệu (giai đoạn 2)

Không cần sinh đủ khối lượng thật. Đề chỉ yêu cầu **≥ 70.000 dòng** cho bảng cần minh họa. Đề xuất:

| Bảng | Số dòng sinh | Ghi chú |
|---|---|---|
| `CHI_NHANH` / `NHAN_VIEN` / `LICH_SU_CONG_TAC` / `HOA_HONG_CHI_NHANH` | 12 / ~700 / ~1.000 / ~24 | Nhỏ, đúng như thực tế |
| `HANG_MUC_CUNG_CAP` / `TON_KHO` | ~180 / ~1.800 | |
| `HOI_VIEN` | ~120.000 | |
| `GOI_HOI_VIEN` | ~200.000 | Mỗi hội viên vài gói theo thời gian |
| `CHECK_IN` | ~2.000.000 (khoảng 3 tháng) | Đủ để partition theo tháng có ý nghĩa |
| `LICH_HEN_PT` / `DANH_GIA_HLV` | ~260.000 (3 tháng) / ~100.000 | |
| `HOA_DON` / `CHI_TIET_HOA_DON` | ~500.000 / ~750.000 (1 năm) | Đủ một năm để minh họa hạng theo năm |

## Nguồn tham khảo

1. [What gym check-ins reveal about member behavior — Wellyx](https://wellyx.com/blog/what-gym-check-ins-reveal-about-member-behavior/)
2. [When South Africa Goes to Gym: 2.4 Million Check-Ins Analysed — Itensity](https://itensityonline.com/?p=948336)
3. [Gym membership statistics (số liệu IHRSA) — Wod.guru](https://wod.guru/blog/en/gym-membership-statistics/)
4. [IHRSA Fitness Training Report Shows Consumer Preferences — Health & Fitness Association](https://www.healthandfitness.org/ihrsa-fitness-training-report-shows-consumer-preferences/)
5. [What Does a Personal Trainer Do on a Daily Basis? — FitSW](https://www.fitsw.com/blog/what-does-a-personal-trainer-do-on-a-daily-basis/)

Các nguồn là số liệu ngành ở Mỹ/thế giới, phần lớn từ 2014–2016; chưa tìm được số liệu công khai của chuỗi gym Việt Nam. Dòng "Tự đặt" là giả định của nhóm, có thể đổi.
