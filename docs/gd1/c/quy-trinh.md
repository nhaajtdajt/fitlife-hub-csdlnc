# Mô tả quy trình — Module C: Check-in, lịch hẹn PT và đánh giá HLV

## 1. Mục tiêu và phạm vi

Module C ghi nhận lượt vào/ra phòng tập, quản lý lịch hẹn PT và đánh giá HLV. Vấn đề của đề cần giải:
- **Vấn đề 1 — nghẽn check-in giờ cao điểm:** tra cứu quyền vào cổng dưới 50 ms.
- **Vấn đề 2 — xung đột đặt lịch PT:** ngăn tuyệt đối hai lịch trùng một PT cùng khung giờ.

Dữ liệu theo [lược đồ logic](../luoc-do-logic.png).

## 2. Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Hội viên | Quét mã vào/ra, đặt/hủy lịch PT, đánh giá HLV |
| Lễ tân | Đặt lịch thay hội viên, tra cứu người đang có mặt và lịch sử tập |
| PT | Xem lịch dạy |
| Quản lý chi nhánh | Xem hiệu suất HLV |
| Hệ thống | Xác thực quyền vào, chống trùng lịch, trừ buổi PT tới giờ, đóng phiên cuối ngày |

| Bảng | Nội dung chính |
|---|---|
| `CHECK_IN` | Gói đã quét (`MaGHV`), chi nhánh, thời gian vào/ra, thiết bị vào/ra |
| `LICH_HEN_PT` | Gói PT (`MaGHV`), PT (`MaNV`), chi nhánh, ngày tập, giờ bắt đầu, trạng thái |
| `DANH_GIA_HLV` | Một lịch PT, điểm, nhận xét |
| Dữ liệu B | `GOI_HOI_VIEN`, `HANG_MUC_CUNG_CAP` để kiểm tra quyền và số buổi |
| Dữ liệu A | `CHI_NHANH` (giờ mở/đóng cửa), `LICH_SU_CONG_TAC` để kiểm tra PT |

## 3. Quy tắc nghiệp vụ chung

**Check-in**

1. Hội viên quét **mã QR của một gói** (`MaGHV`), không quét mã hội viên. Hội viên chọn gói nào để quét thì hệ thống dùng đúng gói đó.
2. Gói được vào cổng khi: `NgayKichHoat <= ngày quét <= NgayHetHan`, `SoBuoiConLai` rỗng hoặc > 0, và loại hàng mục **khác PT**. Gói dùng được ở mọi chi nhánh.
3. Quét vào bằng gói Spa hoặc vé lượt thì trừ 1 lượt trong cùng giao dịch tạo check-in.
4. Quét vào **không** kiểm tra hội viên có đang có phiên mở hay không, để cổng nhanh nhất.
5. Quét ra đóng **mọi** phiên đang mở của hội viên. Phiên mới nhất ghi thiết bị ra thật; các phiên cũ hơn để `ThietBiRa` rỗng.
6. Cuối ngày, phiên còn mở được đóng tại giờ đóng cửa chi nhánh, `ThietBiRa` rỗng.
7. `ThoiGianRa` có giá trị mà `ThietBiRa` rỗng nghĩa là **phiên bị tự đóng**, giờ ra là ước tính. `ThoiGianRa` luôn sau `ThoiGianVao`.
8. Cổng không chặn theo sức chứa. Thiết bị cổng chỉ hoạt động trong giờ mở cửa chi nhánh; quét ngoài giờ nằm ngoài phạm vi.
9. **Ngoài phạm vi:** chống dùng chung mã (cho người khác mượn mã QR); phân khu Gym/Yoga trong chi nhánh. Hệ quả được chấp nhận: hội viên ra không quét rồi vào lại có thể có hai phiên mở cùng lúc, nên báo cáo lấp đầy có thể đếm dư.

**Lịch hẹn PT**

10. Mỗi buổi kéo dài một giờ, bắt đầu vào **giờ tròn** (phút và giây bằng 0), và nằm trọn trong giờ mở cửa: `ThoiGianBatDau >= GioMoCua` và `ThoiGianBatDau + 1 giờ <= GioDongCua`.
11. `TrangThai` không rỗng, gồm: **0 = Đã đặt** (mặc định), **1 = Hủy**, **2 = Đã hoàn thành**. Chỉ có ba đường chuyển: 0 → 1 (hội viên hủy đúng hạn), 0 → 2 (job tới giờ). Không có đường quay lại 0, không chuyển giữa 1 và 2.
12. **Chống trùng lịch PT:** trong các lịch có `TrangThai <> 1`, bộ `(MaNV, NgayTap, ThoiGianBatDau)` là duy nhất. Ràng buộc này đặt tại CSDL, nên hai giao dịch đặt đồng thời thì chỉ một thành công.
13. Một hội viên không có hai lịch chưa hủy trùng ngày, trùng giờ bắt đầu, dù khác PT.
14. Nhân viên của lịch phải có chức vụ PT (`NHAN_VIEN.ChucVu`, cố định) và có giai đoạn công tác tại đúng `MaCN` của lịch, bao phủ `NgayTap`. Xét cả giai đoạn đang chờ hiệu lực, vì A có thể đã chấp nhận điều chuyển từ ngày mai.
15. Gói của lịch phải là gói PT của hội viên, còn hạn tại `NgayTap` và còn buổi trống (quy tắc chọn gói ở QT-B3).
16. Hủy chỉ được khi còn **ít nhất 2 giờ** trước giờ tập. Hủy thì không trừ buổi.
17. Tới giờ tập, lịch còn Đã đặt được chuyển sang Đã hoàn thành và trừ 1 buổi, **kể cả hội viên không đến**. PT vẫn được tính hoa hồng buổi đó, vì khung giờ đã bị giữ.
18. Chỉ lịch Đã hoàn thành được đánh giá; mỗi lịch tối đa một đánh giá; điểm nguyên từ 1 đến 5.
19. Lịch có `NgayTap` thuộc tháng đã qua không được sửa hoặc xóa, để hoa hồng tháng cũ không đổi.

## 4. QT-C1 — Quét vào

**Ai làm:** hội viên tại cổng vào.

1. Thiết bị gửi `MaGHV` đọc từ mã QR, `MaCN` và tên thiết bị của cổng, thời điểm quét.
2. Hệ thống đọc gói theo `MaGHV` (tra theo khóa chính) và loại hàng mục của gói.
3. Kiểm tra gói theo quy tắc 2.
4. Trong một giao dịch: nếu gói là Spa hoặc vé lượt, giảm `SoBuoiConLai` từ 1 về 0 (chỉ khi vẫn còn 1); tạo `CHECK_IN` với `ThoiGianVao` = thời điểm quét, `ThietBiVao`, `ThoiGianRa` rỗng.
5. Mở cổng.

**Ngoại lệ:** mã không tồn tại; gói chưa kích hoạt, hết hạn, hết lượt hoặc là gói PT; vé vừa bị dùng bởi một lần quét khác cùng lúc. Mọi lỗi đều không mở cổng, không tạo check-in, không trừ lượt.

**Hiệu năng:** đường xử lý chỉ đọc một gói theo khóa chính và ghi một dòng; báo cáo không chạy trong giao dịch này. Mục tiêu dưới 50 ms sẽ đo ở GĐ2.

## 5. QT-C2 — Quét ra

**Ai làm:** hội viên tại cổng ra.

1. Thiết bị gửi mã QR (gói nào của hội viên cũng được, kể cả gói đã hết hạn), tên thiết bị, thời điểm quét.
2. Hệ thống xác định hội viên từ gói, rồi tìm mọi `CHECK_IN` đang mở (`ThoiGianRa` rỗng) của các gói thuộc hội viên đó.
3. Trong một giao dịch: ghi `ThoiGianRa` = thời điểm quét cho tất cả các phiên đó; phiên có `ThoiGianVao` mới nhất ghi `ThietBiRa`, các phiên còn lại để `ThietBiRa` rỗng.
4. Mở cổng.

**Ngoại lệ:** không có phiên nào đang mở (ví dụ đã bị tự đóng) → vẫn mở cổng ra, không ghi gì. Mã không tồn tại → không ghi, lễ tân hỗ trợ.

## 6. QT-C3 — Đóng phiên quên quét ra cuối ngày

**Ai làm:** job hệ thống, chạy cho từng chi nhánh sau giờ đóng cửa.

1. Lấy các `CHECK_IN` tại chi nhánh có `ThoiGianVao` trong ngày và `ThoiGianRa` rỗng.
2. Ghi `ThoiGianRa` = ngày đó + `GioDongCua` của chi nhánh, `ThietBiRa` rỗng. Chỉ ghi nếu `ThoiGianRa` vẫn còn rỗng lúc ghi.
3. Ghi lại số phiên đã đóng.

**Ngoại lệ:** job lỗi hoặc bỏ lỡ một ngày → chạy bù theo **giờ đóng cửa của chính ngày đó**, không dùng thời điểm chạy bù. Chạy lại không ghi đè phiên đã đóng.

## 7. QT-C4 — Tra cứu người đang có mặt

**Ai làm:** lễ tân, quản lý chi nhánh.

1. Chọn chi nhánh (mặc định chi nhánh mình) và thời điểm T (mặc định bây giờ).
2. Hệ thống liệt kê các check-in tại chi nhánh có `ThoiGianVao <= T` và (`ThoiGianRa` rỗng hoặc `ThoiGianRa > T`), kèm hội viên (qua gói) và giờ vào.
3. Hiển thị số người và so với sức chứa để tham khảo.

**Ghi chú:** theo quy tắc 9, một hội viên có thể xuất hiện hai lần nếu đã ra không quét rồi vào lại. Không có ai thì trả danh sách rỗng.

## 8. QT-C5 — Xem lịch trống và đặt lịch PT

**Ai làm:** hội viên, hoặc lễ tân thao tác thay.

1. Chọn chi nhánh, ngày tập, có thể chọn PT.
2. Hệ thống lấy các PT (nhân viên có chức vụ PT) có giai đoạn công tác tại chi nhánh đó bao phủ ngày tập (quy tắc 14).
3. Với mỗi PT, liệt kê các khung giờ tròn trong giờ mở cửa (quy tắc 10), bỏ các khung đã có lịch chưa hủy.
4. Hội viên chọn PT và khung giờ.
5. Hệ thống chọn gói PT của hội viên theo QT-B3 (còn hạn tại ngày tập, còn buổi trống, hết hạn sớm nhất trước).
6. Trong một giao dịch, kiểm tra lại trên dữ liệu mới nhất: PT vẫn hợp lệ, gói vẫn còn buổi trống, hội viên chưa có lịch khác cùng giờ (quy tắc 13), giờ tập còn ở tương lai. Tạo `LICH_HEN_PT` với `TrangThai` = 0.
7. Nếu ràng buộc chống trùng (quy tắc 12) báo khung vừa bị người khác đặt → báo hội viên chọn khung khác.

**Ngoại lệ:** không có gói PT phù hợp; PT không làm tại chi nhánh vào ngày đó; khung đã kín; khung ngoài giờ mở cửa hoặc không phải giờ tròn; giờ tập đã qua; hội viên đã có lịch cùng giờ. Mọi lỗi đều không tạo lịch.

## 9. QT-C6 — Hủy lịch PT

**Ai làm:** hội viên, hoặc lễ tân thao tác thay.

1. Chọn lịch đang ở trạng thái Đã đặt của chính hội viên (xác định qua gói của lịch).
2. Kiểm tra thời điểm hủy <= ngày giờ tập − 2 giờ (đúng mốc 2 giờ vẫn được hủy).
3. Chuyển `TrangThai` = 1. Buổi đã giữ được trả lại gói vì không còn tính là lịch Đã đặt.

**Ngoại lệ:** còn dưới 2 giờ → từ chối, lịch giữ nguyên và tới giờ vẫn bị trừ. Lịch đã hủy, đã hoàn thành, hoặc không phải của hội viên → từ chối.

## 10. QT-C7 — Tự hoàn thành lịch và trừ buổi khi tới giờ tập

**Ai làm:** job hệ thống, chạy định kỳ (ví dụ mỗi đầu giờ).

1. Lấy các lịch có `TrangThai` = 0 và ngày giờ tập <= bây giờ, kể cả lịch bị bỏ lỡ ở lần chạy trước.
2. Với từng lịch, trong một giao dịch: chuyển `TrangThai` = 2 và giảm `SoBuoiConLai` của đúng gói `MaGHV` của lịch đi 1. Chỉ cập nhật nếu lịch vẫn còn trạng thái 0 lúc ghi.
3. Không đọc check-in: hội viên có đến hay không đều bị trừ (quy tắc 17).

**Ngoại lệ:** lỗi → hoàn tác lịch đó, lần chạy sau xử lý lại. Lịch đã chuyển trạng thái thì không bao giờ bị trừ lần hai. Số buổi của gói không bao giờ âm, vì lịch chỉ được tạo khi còn buổi trống.

## 11. QT-C8 — Tra cứu lịch sử tập và lịch PT

**Ai làm:** hội viên, lễ tân; PT xem lịch dạy của mình.

1. **Hội viên / lễ tân:** chọn hội viên và khoảng ngày. Hệ thống liệt kê các check-in (qua các gói của hội viên) và các lịch PT trong khoảng, kèm chi nhánh, PT, trạng thái.
2. **PT:** chọn khoảng ngày, xem các lịch của mình kèm hội viên, chi nhánh, trạng thái.

Không có dữ liệu thì trả danh sách rỗng.

## 12. QT-C9 — Đánh giá HLV

**Ai làm:** hội viên của buổi PT.

1. Chọn một lịch Đã hoàn thành của mình chưa được đánh giá.
2. Nhập điểm nguyên 1–5 và nhận xét (tùy chọn).
3. Hệ thống kiểm tra chủ sở hữu (qua gói của lịch), trạng thái và chưa có đánh giá, rồi tạo `DANH_GIA_HLV`.

**Ngoại lệ:** lịch chưa hoàn thành hoặc đã hủy; không phải lịch của hội viên; điểm ngoài 1–5; lịch đã có đánh giá.

## 13. QT-C10 — Xem hiệu suất HLV

**Ai làm:** quản lý chi nhánh, ban giám đốc.

1. Chọn khoảng thời gian, chi nhánh, có thể chọn PT.
2. Với mỗi PT: số buổi Đã hoàn thành trong khoảng tại chi nhánh, số đánh giá, điểm đánh giá trung bình.
3. PT chưa có đánh giá thì hiển thị "chưa có đánh giá", không hiển thị điểm 0.

**Ghi chú:** số buổi Đã hoàn thành gồm cả buổi hội viên không đến (quy tắc 17).

## 14. Quan hệ với module khác

| Module | C cung cấp | C sử dụng |
|---|---|---|
| A | `LICH_HEN_PT(MaLH, MaNV, MaCN, NgayTap, ThoiGianBatDau, TrangThai)` để tính hoa hồng và kiểm tra lịch vướng | `CHI_NHANH(MaCN, GioMoCua, GioDongCua)`; `NHAN_VIEN(MaNV, ChucVu)`; `LICH_SU_CONG_TAC(MaNV, MaCN, NgayBatDau, NgayKetThuc)` |
| B | Giảm `SoBuoiConLai` khi dùng vé và khi trừ buổi PT | `GOI_HOI_VIEN(MaGHV, MaHV, MaHM, NgayKichHoat, NgayHetHan, SoBuoiConLai)`; `HANG_MUC_CUNG_CAP(MaHM, LoaiHangMuc)` |
| D | `CHECK_IN(MaCN, ThoiGianVao, ThoiGianRa)` cho báo cáo lấp đầy | — |
