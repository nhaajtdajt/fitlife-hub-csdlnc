# Mô tả quy trình — Module D: Sản phẩm, tồn kho, hóa đơn và báo cáo

## 1. Mục tiêu và phạm vi

Module D quản lý sản phẩm bán lẻ, tồn kho theo chi nhánh, hóa đơn và các báo cáo. Vấn đề của đề cần giải: **vấn đề 3 — dữ liệu log lớn, báo cáo doanh thu và tỷ lệ lấp đầy chậm.**

Dữ liệu theo [lược đồ logic](../luoc-do-logic.png).

## 2. Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Nhân viên bán hàng / lễ tân | Lập và thanh toán hóa đơn |
| Nhân viên kho / quản lý chi nhánh | Nhập hàng, điều chỉnh tồn, mở/ngừng bán sản phẩm tại chi nhánh |
| Quản lý chi nhánh | Xem doanh thu, bán chạy, tồn kho, lấp đầy của chi nhánh mình |
| Ban giám đốc / kế toán | Quản lý danh mục sản phẩm, xem báo cáo toàn hệ thống, đối chiếu hóa đơn |
| Hệ thống | Tính tiền, kiểm tra tồn, ghi hóa đơn, trừ kho, gọi B tạo gói và cộng điểm |

| Bảng | Nội dung chính |
|---|---|
| `HANG_MUC_CUNG_CAP` | Danh mục chung với B. D quản lý các dòng loại **Sản phẩm** |
| `TON_KHO` | Chi nhánh, sản phẩm, số lượng tồn |
| `HOA_DON` | Hội viên, nhân viên lập, chi nhánh, ngày lập, giảm giá, tổng tiền, hình thức thanh toán |
| `CHI_TIET_HOA_DON` | Hóa đơn, hàng mục (gói hoặc sản phẩm), số lượng, đơn giá lúc bán |
| `CHECK_IN` (C) | Nguồn dữ liệu cho báo cáo lấp đầy, D chỉ đọc |

## 3. Quy tắc nghiệp vụ chung

1. Mỗi hóa đơn thuộc một hội viên, một nhân viên lập và một chi nhánh. **Chi nhánh của hóa đơn là chi nhánh hiện tại của nhân viên lập** (`NHAN_VIEN.MaCNHienTai`), không chọn tự do. Khách chưa là hội viên phải đăng ký (QT-B1) trước khi mua.
2. Hóa đơn có ít nhất một dòng. Mỗi dòng là một hàng mục (gói hoặc sản phẩm). Trong một hóa đơn, mỗi hàng mục xuất hiện tối đa một dòng; mua nhiều thì tăng `SoLuong`.
3. `SoLuong` nguyên dương. `DonGia` là giá của hàng mục **tại lúc bán**; sửa giá danh mục về sau không đổi hóa đơn cũ.
4. Thành tiền dòng = `SoLuong × DonGia`, tính khi đọc, không lưu. Tạm tính = tổng thành tiền các dòng. `0 <= GiamGia <= tạm tính`. `TongTien = tạm tính − GiamGia`, được lưu sẵn trên `HOA_DON` để báo cáo và cộng điểm nhanh.
5. `HinhThucThanhToan` thuộc {Tiền mặt, VNPay, Thẻ}. Chỉ thanh toán thành công mới ghi nhận hóa đơn.
6. Thanh toán là **một giao dịch nguyên tử**: tạo hóa đơn, các dòng, trừ tồn kho cho dòng sản phẩm, B tạo gói cho dòng gói (QT-B2), B cộng điểm/hạng (QT-B4). Lỗi ở bất kỳ bước nào thì hoàn tác toàn bộ.
7. Hóa đơn đã thanh toán không sửa, không xóa, không hoàn tiền. Sai sót chỉ sửa được trước khi thanh toán.
8. **Tồn kho:** có dòng `TON_KHO(MaCN, MaHM)` nghĩa là chi nhánh đó bán sản phẩm đó. `SoLuongTon >= 0`. Hết hàng thì tồn bằng 0 và vẫn giữ dòng. Chỉ hàng mục loại Sản phẩm mới có dòng tồn kho; gói có ở mọi chi nhánh.
9. Chỉ lưu số tồn hiện tại; nhật ký từng lần nhập/xuất kho nằm ngoài phạm vi. Lượng đã bán vẫn tra được từ `CHI_TIET_HOA_DON`.
10. Không xóa hàng mục đã xuất hiện trên hóa đơn hoặc tồn kho; chuyển `DangKinhDoanh` = 0.
11. **Doanh thu:** doanh thu thực thu (tổng, theo chi nhánh, theo thời gian) cộng `TongTien`, tức là sau giảm giá. Doanh thu **theo từng hàng mục** tính theo giá niêm yết lúc bán, `SoLuong × DonGia`, trước giảm giá, vì giảm giá chỉ có ở cấp hóa đơn. Vì vậy tổng doanh thu các hàng mục có thể lớn hơn doanh thu thực thu; báo cáo phải ghi rõ "trước giảm giá".

## 4. QT-D1 — Quản lý danh mục sản phẩm

**Ai làm:** ban giám đốc / quản lý được phân quyền.

1. Thêm sản phẩm vào `HANG_MUC_CUNG_CAP` với `LoaiHangMuc` = Sản phẩm: tên (ghi rõ nhóm như Whey protein, bình nước, thảm yoga), giá bán.
2. Sửa tên, giá, hoặc chuyển ngừng kinh doanh.
3. Hệ thống kiểm tra: tên bắt buộc, giá dương, `ThoiHanThang` và `SoBuoiBanDau` rỗng.

**Ngoại lệ:** thiếu tên, giá không dương, có thời hạn hoặc số buổi → nhập lại. Xóa sản phẩm đã có giao dịch → từ chối, chỉ ngừng kinh doanh.

## 5. QT-D2 — Mở hoặc ngừng bán sản phẩm tại chi nhánh

**Ai làm:** quản lý chi nhánh.

1. **Mở bán:** chọn sản phẩm đang kinh doanh, nhập số lượng kiểm kê ban đầu (>= 0). Hệ thống tạo dòng `TON_KHO` cho chi nhánh mình.
2. **Ngừng bán tại chi nhánh:** chỉ được khi `SoLuongTon` = 0; hệ thống xóa dòng `TON_KHO`. Hóa đơn cũ không bị ảnh hưởng vì dòng hóa đơn tham chiếu hàng mục, không tham chiếu tồn kho.

**Ngoại lệ:** sản phẩm đã có dòng tồn tại chi nhánh; hàng mục không phải loại Sản phẩm; ngừng bán khi còn hàng → từ chối.

## 6. QT-D3 — Nhập hàng và điều chỉnh tồn kho

**Ai làm:** nhân viên kho / quản lý chi nhánh.

1. Chọn sản phẩm đang bán tại chi nhánh, chọn nhập hàng (số dương) hoặc kiểm kê điều chỉnh (số dương hoặc âm).
2. Hệ thống cộng số lượng vào `SoLuongTon` trong một giao dịch, kiểm tra kết quả không âm.
3. Hiển thị số tồn mới.

**Ngoại lệ:** chi nhánh chưa mở bán sản phẩm đó; số lượng không phải số nguyên; tồn sau điều chỉnh âm → từ chối, không ghi.

## 7. QT-D4 — Lập và thanh toán hóa đơn

**Ai làm:** nhân viên bán hàng / lễ tân.

1. Nhân viên đăng nhập; hệ thống lấy chi nhánh hiện tại của nhân viên làm chi nhánh hóa đơn.
2. Tìm hội viên theo mã hoặc SĐT; chưa có thì làm QT-B1.
3. Thêm các dòng gói và/hoặc sản phẩm. Với mỗi dòng: hàng mục đang kinh doanh, số lượng nguyên dương, đơn giá lấy từ giá hiện tại. Dòng sản phẩm phải có dòng tồn kho tại chi nhánh.
4. Với dòng gói, hệ thống hiển thị ngày kích hoạt, ngày hết hạn và cảnh báo nếu chồng với gói cùng loại còn hạn (QT-B2).
5. Hệ thống tính tạm tính, nhân viên nhập giảm giá, hệ thống tính `TongTien`. Chọn hình thức thanh toán.
6. Kiểm tra tồn kho mọi dòng sản phẩm đủ hàng.
7. Khi thanh toán thành công, trong một giao dịch:
   1. tạo `HOA_DON` và các `CHI_TIET_HOA_DON`;
   2. trừ `SoLuongTon` của từng dòng sản phẩm tại chi nhánh, chỉ khi vẫn đủ hàng lúc ghi;
   3. B tạo `GOI_HOI_VIEN` cho từng đơn vị gói (QT-B2);
   4. B cộng `TongChiTieu`, `DiemTichLuy` và cập nhật `Hang` (QT-B4).
8. In biên nhận.

**Ngoại lệ:** thanh toán bị từ chối; hàng mục ngừng kinh doanh; thiếu tồn (kể cả do hóa đơn khác vừa bán hết); giảm giá âm hoặc lớn hơn tạm tính; nhân viên đã nghỉ việc; B tạo gói hoặc cộng điểm lỗi. Mọi lỗi đều hoàn tác toàn bộ: không có hóa đơn thiếu dòng, không trừ kho một phần, không có tiền mà thiếu gói hoặc điểm.

## 8. QT-D5 — Tra cứu và đối chiếu hóa đơn

**Ai làm:** nhân viên bán hàng, kế toán, quản lý theo quyền.

1. Lọc theo mã hóa đơn, hội viên, khoảng ngày, chi nhánh hoặc nhân viên lập.
2. Hệ thống trả thông tin hóa đơn và các dòng, gồm đơn giá lúc bán và thành tiền tính từ `SoLuong × DonGia`.
3. Kế toán đối chiếu: tạm tính − `GiamGia` phải bằng `TongTien`.

**Ngoại lệ:** khoảng ngày không hợp lệ → nhập lại. Không tìm thấy → danh sách rỗng. Hóa đơn chỉ xem, không sửa.

## 9. QT-D6 — Báo cáo doanh thu, bán chạy và tồn kho

**Ai làm:** quản lý chi nhánh (chi nhánh mình), ban giám đốc (toàn hệ thống).

- **Doanh thu theo ngày/tháng/quý/năm:** cộng `TongTien` theo `NgayLap` và chi nhánh, mỗi hóa đơn đúng một lần. So sánh giữa các chi nhánh ở cấp công ty.
- **Doanh thu theo hàng mục** (ví dụ gói doanh thu cao nhất 6 tháng gần nhất): cộng `SoLuong × DonGia` của các dòng trong kỳ, nhóm theo `MaHM`, có thể lọc theo loại và chi nhánh. Đây là doanh thu trước giảm giá (quy tắc 11).
- **Bán chạy nhất:** cộng `SoLuong` theo hàng mục trong kỳ, lọc theo chi nhánh và loại.
- **Tồn kho:** số lượng hiện có theo chi nhánh và sản phẩm.

Kỳ không có giao dịch trả 0 hoặc danh sách rỗng. Báo cáo chỉ đọc, không sửa dữ liệu.

## 10. QT-D7 — Báo cáo tỷ lệ lấp đầy

**Ai làm:** quản lý chi nhánh, ban giám đốc.

1. Chọn chi nhánh, ngày và các mốc giờ (ví dụ 17:00, 18:00, 19:00).
2. Tại mỗi mốc T: số người = số `CHECK_IN` tại chi nhánh có `ThoiGianVao <= T` và (`ThoiGianRa` rỗng hoặc `ThoiGianRa > T`). Tỷ lệ = 100 × số người / `SucChua`. Tỷ lệ có thể vượt 100%, không cắt ngọn.
3. Hiển thị bảng tỷ lệ theo từng mốc giờ, có thể so sánh giữa các chi nhánh.

**Ghi chú:** đây là số người có mặt tại đúng mốc T, không phải tổng lượt vào trong cả giờ. Sức chứa chỉ dùng để tính tỷ lệ, không chặn cổng.

## 11. Quan hệ với module khác

| Module | D cung cấp | D sử dụng |
|---|---|---|
| A | — | `CHI_NHANH(MaCN, SucChua)`; `NHAN_VIEN(MaNV, MaCNHienTai)` |
| B | `HOA_DON(MaHD, MaHV, NgayLap, TongTien)`; `CHI_TIET_HOA_DON(MaHM, SoLuong)` để B tạo gói và cộng điểm trong cùng giao dịch | `HOI_VIEN(MaHV)`; danh mục gói trong `HANG_MUC_CUNG_CAP` |
| C | — | `CHECK_IN(MaCN, ThoiGianVao, ThoiGianRa)` cho báo cáo lấp đầy |
