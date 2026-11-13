# Mô tả quy trình — Module B: Hội viên, gói dịch vụ và điểm/hạng

## 1. Mục tiêu và phạm vi

Module B quản lý hội viên, danh mục gói dịch vụ, các gói hội viên đã mua và chương trình hội viên thân thiết (điểm, hạng). Vấn đề của đề cần giải: **vấn đề 4 — tính lại hạng quá tải khi thanh toán**.

Hóa đơn thuộc D; check-in và lịch hẹn PT thuộc C. Dữ liệu theo [lược đồ logic](../luoc-do-logic.png).

## 2. Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Lễ tân | Đăng ký, cập nhật, tra cứu hội viên |
| Nhân viên bán hàng / lễ tân | Bán gói (cùng lúc lập hóa đơn của D) |
| Quản lý chi nhánh | Xem hội viên sắp hết hạn, lâu ngày chưa quay lại |
| Ban giám đốc | Quản lý danh mục gói, xem phân bổ hạng |
| Hệ thống | Tạo gói khi thanh toán, cộng điểm/hạng, reset hạng đầu năm |

| Bảng | Nội dung chính |
|---|---|
| `HOI_VIEN` | Thông tin cá nhân, ngày đăng ký, chi nhánh đăng ký (`MaCN`), tổng chi tiêu trong năm, điểm tích lũy, hạng |
| `HANG_MUC_CUNG_CAP` | Danh mục chung với D. B quản lý các dòng loại **Gym, Yoga, PT, Spa, Vé lượt**; D quản lý loại **Sản phẩm** |
| `GOI_HOI_VIEN` | Mỗi gói đã mua: hội viên, hàng mục, ngày kích hoạt, ngày hết hạn, số buổi/lượt còn lại |

## 3. Quy tắc nghiệp vụ chung

1. Mỗi gói đã mua là một dòng `GOI_HOI_VIEN` với mã `MaGHV` riêng. Mã này cũng là **mã QR** hội viên dùng để quét cổng (C). Hội viên có thể có nhiều gói cùng lúc, kể cả nhiều gói cùng loại.
2. Mọi gói **bắt đầu ngay ngày mua**: `NgayKichHoat` = ngày lập hóa đơn. Không gia hạn nối tiếp. Mua gói mới khi gói cũ còn hạn thì hai gói chạy song song; phần ngày chồng nhau không được bù.
3. Quyền lợi của từng loại khi tạo gói:

   | Loại | `NgayHetHan` | `SoBuoiConLai` |
   |---|---|---|
   | Gym, Yoga | Ngày mua + `ThoiHanThang` tháng − 1 ngày | Rỗng (không giới hạn lượt vào) |
   | PT | Ngày mua + `ThoiHanThang` tháng − 1 ngày | `SoBuoiBanDau`, ví dụ 24 |
   | Spa, Vé lượt | Ngày mua + 6 ngày (hạn 7 ngày) | 1 |

   Cộng tháng mà tháng đích không có ngày tương ứng thì lấy ngày cuối tháng đích, rồi mới trừ 1 ngày. Ví dụ mua gói Gym 1 tháng ngày 12/11/2026 thì dùng tới hết 11/12/2026; mua vé lượt ngày 12/11 thì dùng tới hết 18/11.
4. `NgayHetHan` và `SoBuoiConLai` được **chụp lại lúc mua**. Sửa danh mục về sau không làm đổi gói đã bán.
5. Trạng thái của gói suy ra từ dữ liệu, không lưu riêng: còn hiệu lực khi `NgayKichHoat <= hôm nay <= NgayHetHan` và (`SoBuoiConLai` rỗng hoặc > 0); quá `NgayHetHan` là hết hạn; `SoBuoiConLai = 0` là đã dùng hết.
6. **Quyền vào cổng:** gói còn hiệu lực và **không phải loại PT**. Gói PT chỉ dùng để đặt lịch PT. Mua PT **không** bắt buộc phải có hoặc mua kèm thẻ Gym; muốn vào tập thì hội viên dùng thẻ hoặc mua vé như mọi người.
7. **Điểm:** mỗi hóa đơn được cộng phần nguyên của `TongTien / 100.000` (tổng sau giảm giá). Phần lẻ bỏ, không cộng dồn. Điểm không hết hạn, không reset, chưa hỗ trợ đổi quà.
8. **Hạng** theo tổng chi tiêu **trong năm dương lịch** (`TongChiTieu`):

   | Hạng | Chi tiêu trong năm |
   |---|---|
   | Silver | dưới 20 triệu |
   | Gold | từ 20 triệu đến dưới 50 triệu |
   | Platinum | từ 50 triệu |

   Ngưỡng do nhóm quyết định. Trong năm tổng chi tiêu chỉ tăng nên hạng chỉ tăng. Đầu năm mới toàn bộ hội viên về 0 và Silver (QT-B5).
9. Hóa đơn đã thanh toán không sửa, không xóa, không hoàn tiền; gói đã tạo và điểm đã cộng giữ nguyên.
10. Bảo lưu gói và chuyển nhượng gói nằm ngoài phạm vi.

## 4. QT-B1 — Đăng ký và cập nhật hội viên

**Ai làm:** lễ tân tại bất kỳ chi nhánh nào.

**Đăng ký:**
1. Hội viên cung cấp họ tên, SĐT, email, CCCD, giới tính, ngày sinh.
2. Lễ tân tìm theo CCCD và SĐT để chắc chắn chưa có hồ sơ.
3. Hệ thống tạo `HOI_VIEN`: `NgayDangKy` = hôm nay, `MaCN` = chi nhánh hiện tại của lễ tân, `TongChiTieu` = 0, `DiemTichLuy` = 0, `Hang` = Silver.

**Cập nhật:** tìm theo mã, SĐT hoặc CCCD rồi sửa thông tin liên hệ. Không sửa trực tiếp tổng chi tiêu, điểm, hạng. Hội viên nghỉ tập vẫn giữ hồ sơ vì còn hóa đơn và gói tham chiếu.

**Ngoại lệ:**
- CCCD đã có hồ sơ → không tạo mới, dùng lại hồ sơ cũ.
- SĐT đã thuộc hồ sơ khác → từ chối, vì SĐT dùng để tra cứu.
- Thiếu thông tin hoặc sai định dạng (CCCD không đủ 12 số, ngày sinh trong tương lai) → nhập lại.

## 5. QT-B2 — Mua gói

**Ai làm:** nhân viên bán hàng / lễ tân, trong lúc lập hóa đơn của D (QT-D3).

1. Tìm hội viên; chưa có hồ sơ thì làm QT-B1 trước.
2. Chọn một hay nhiều gói đang kinh doanh (`DangKinhDoanh` = 1). Gói dùng được ở mọi chi nhánh.
3. Hệ thống hiển thị ngày kích hoạt và ngày hết hạn của từng gói. Nếu hội viên đang có gói cùng loại còn hạn, hiển thị cảnh báo phần ngày sẽ chồng nhau để hội viên cân nhắc.
4. Khi thanh toán thành công, **trong cùng giao dịch với hóa đơn**: mỗi dòng gói có số lượng n tạo đúng n dòng `GOI_HOI_VIEN` theo quy tắc 3; sau đó cập nhật điểm/hạng theo QT-B4.

**Ngoại lệ:**
- Thanh toán thất bại → không có hóa đơn, không có gói, không cộng điểm.
- Gói đã ngừng kinh doanh → không cho chọn.
- Tạo gói lỗi → hoàn tác cả hóa đơn.

**Ví dụ:** gói Gym cũ hết hạn 15/11/2026, ngày 08/11 hội viên mua thêm Gym 1 tháng. Gói mới dùng từ 08/11 đến hết 07/12; từ 08/11 đến 15/11 hai gói chạy song song.

## 6. QT-B3 — Sử dụng quyền lợi gói

**Ai làm:** hội viên; hệ thống của C ghi nhận.

**Vào cổng (C thực hiện, B quy định quyền):**
- Hội viên quét mã QR của một gói. Gói phải còn hiệu lực và không phải PT.
- Gói Spa hoặc vé lượt: `SoBuoiConLai` giảm từ 1 về 0 trong cùng giao dịch tạo check-in. Ra rồi muốn vào lại cần gói khác còn hiệu lực.
- Gói Gym/Yoga: không trừ gì.

**Gói PT:**
1. **Đặt lịch:** hệ thống chọn gói PT của hội viên còn hạn tại ngày tập và còn buổi trống. Số buổi trống của một gói = `SoBuoiConLai` − số lịch **Đã đặt** của gói đó. Có nhiều gói phù hợp thì ưu tiên gói hết hạn sớm nhất, cùng hạn thì `MaGHV` nhỏ hơn. Lịch lưu `MaGHV` của gói đã chọn và không đổi gói về sau.
2. **Hủy:** hội viên hủy được khi còn **ít nhất 2 giờ** trước giờ tập. Lịch chuyển sang **Hủy**, buổi được trả lại, không trừ.
3. **Đến giờ tập:** job của C chuyển lịch **Đã đặt** sang **Đã hoàn thành** và giảm `SoBuoiConLai` của đúng gói đó đi 1, trong cùng giao dịch. Hội viên không đến cũng bị trừ, vì đã không hủy đúng hạn.
4. Lịch đã chuyển trạng thái thì job không xử lý nữa, nên mỗi lịch bị trừ đúng một lần.

**Ngoại lệ:**
- Không còn gói PT đủ hạn và còn buổi trống → không đặt được.
- Quét mã gói PT ở cổng → cổng từ chối.
- Hai lần đặt đồng thời không được cùng lấy buổi trống cuối cùng của một gói; số buổi không bao giờ âm.

## 7. QT-B4 — Cộng điểm và cập nhật hạng khi thanh toán

**Ai làm:** hệ thống, trong cùng giao dịch thanh toán của D.

**Hiện trạng (vấn đề 4 của đề):** mỗi lần thanh toán, hệ thống cộng lại `TongTien` mọi hóa đơn 12 tháng qua của hội viên để xét hạng. Với ~1.400 hóa đơn/ngày trên bảng hóa đơn ~500.000 dòng/năm, việc này làm chậm in hóa đơn.

**Thiết kế mới:** lưu sẵn `TongChiTieu`, `DiemTichLuy`, `Hang` trên `HOI_VIEN` và chỉ cộng thêm:
1. Nhận `MaHV`, `TongTien` của hóa đơn vừa lập.
2. `TongChiTieu` += `TongTien`.
3. `DiemTichLuy` += phần nguyên `TongTien / 100.000`.
4. So `TongChiTieu` mới với ngưỡng để cập nhật `Hang`.
5. Ghi cùng giao dịch với hóa đơn. Chỉ đọc và sửa đúng một dòng hội viên, không quét lại bảng hóa đơn.

**Ví dụ:** hội viên đang có 18 triệu trong năm, mua 3 triệu → `TongChiTieu` = 21 triệu, lên Gold ngay trên hóa đơn này; được cộng 30 điểm.

**Ngoại lệ:** cập nhật lỗi → hoàn tác cả hóa đơn, không để có hóa đơn đã thu tiền mà thiếu điểm.

## 8. QT-B5 — Reset hạng đầu năm

**Ai làm:** job hệ thống, 00:00 ngày 1/1 hằng năm.

1. Với mọi hội viên, đặt `TongChiTieu` = tổng `TongTien` các hóa đơn của hội viên có `NgayLap` từ 1/1 năm mới (bình thường bằng 0).
2. Tính lại `Hang` theo tổng đó (bình thường về Silver).
3. Không đổi `DiemTichLuy`.

Tính lại từ hóa đơn của năm mới thay vì đặt cứng về 0, để job chạy trễ hoặc chạy lại vẫn đúng: hóa đơn đã thanh toán trong ngày 1/1 trước khi job chạy không bị mất.

**Ngoại lệ:** job lỗi giữa chừng → chạy lại toàn bộ, cho cùng kết quả.

## 9. QT-B6 — Quản lý danh mục gói

**Ai làm:** ban giám đốc.

1. Thêm gói mới vào `HANG_MUC_CUNG_CAP`: tên, loại (Gym, Yoga, PT, Spa, Vé lượt), giá, thời hạn tháng, số buổi ban đầu.
2. Sửa giá hoặc chuyển ngừng kinh doanh (`DangKinhDoanh` = 0). Không xóa gói đã có người mua.
3. Giá mới chỉ áp dụng cho hóa đơn sau đó; hóa đơn cũ giữ đơn giá riêng, gói đã bán giữ quyền lợi đã chụp.

**Ngoại lệ (kiểm tra miền):**
- Giá không dương.
- Gym/Yoga: phải có `ThoiHanThang` thuộc {1, 3, 6, 12}, `SoBuoiBanDau` rỗng.
- PT: `ThoiHanThang` thuộc {3, 6, 12} và `SoBuoiBanDau` > 0.
- Spa, Vé lượt: `ThoiHanThang` rỗng, `SoBuoiBanDau` = 1.

## 10. QT-B7 — Theo dõi hội viên và phân bổ hạng

**Quản lý chi nhánh**, mỗi ngày một lần, xem hội viên **đăng ký tại chi nhánh mình** (`HOI_VIEN.MaCN`):
- **Sắp hết hạn:** có gói Gym/Yoga còn hiệu lực hết hạn trong 7 ngày tới, và không có gói Gym/Yoga nào khác còn hiệu lực sau ngày đó.
- **Lâu ngày chưa quay lại:** còn gói Gym/Yoga hiệu lực nhưng không có check-in nào trong 30 ngày qua (check-in tìm qua các gói của hội viên).

Hai ngưỡng 7 và 30 ngày do nhóm tự đặt. Không có ai thì trả danh sách rỗng.

**Ban giám đốc** xem tỷ lệ Silver / Gold / Platinum toàn hệ thống từ cột `Hang`.

## 11. QT-B8 — Tra cứu hội viên và lịch sử điểm

**Ai làm:** lễ tân, hội viên.

1. Tìm hội viên theo mã, SĐT hoặc CCCD.
2. Hiển thị thông tin, hạng, tổng chi tiêu trong năm, điểm tích lũy và các gói còn hiệu lực.
3. Lịch sử điểm: liệt kê các hóa đơn của hội viên kèm điểm của từng hóa đơn (phần nguyên `TongTien / 100.000`). Không cần bảng lịch sử điểm riêng, vì hóa đơn đã thanh toán không đổi.

## 12. Quan hệ với module khác

| Module | B cung cấp | B sử dụng |
|---|---|---|
| A | — | `CHI_NHANH(MaCN)` cho chi nhánh đăng ký; `NHAN_VIEN(MaCNHienTai)` của lễ tân |
| C | `GOI_HOI_VIEN(MaGHV, MaHV, MaHM, NgayKichHoat, NgayHetHan, SoBuoiConLai)`; `HANG_MUC_CUNG_CAP(MaHM, LoaiHangMuc)` để kiểm tra quyền vào cổng và đặt lịch PT | C giảm `SoBuoiConLai` khi dùng vé và khi trừ buổi PT |
| D | Danh mục gói trong `HANG_MUC_CUNG_CAP`; tạo gói và cập nhật điểm/hạng trong giao dịch thanh toán | `HOA_DON(MaHD, MaHV, NgayLap, TongTien)`; `CHI_TIET_HOA_DON(MaHM, SoLuong)` |
