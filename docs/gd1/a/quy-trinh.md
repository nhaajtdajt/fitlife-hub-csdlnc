# Mô tả quy trình — Module A: Chi nhánh, nhân sự và hoa hồng

## 1. Mục tiêu và phạm vi

Module A quản lý chi nhánh, nhân viên, lịch sử công tác và mức hoa hồng của từng chi nhánh. Câu hỏi trọng tâm (vấn đề 5 của đề): **tại ngày một buổi PT diễn ra, PT thuộc chi nhánh nào và mức hoa hồng của chi nhánh đó là bao nhiêu?** Nhờ lưu lịch sử theo khoảng ngày, hoa hồng được tính đúng kể cả khi PT điều chuyển giữa tháng hoặc chi nhánh đổi mức hoa hồng.

Dữ liệu theo [lược đồ logic](../luoc-do-logic.png).

## 2. Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Quản trị hệ thống | Cập nhật thông tin chi nhánh |
| Quản lý nhân sự | Tiếp nhận, cập nhật, điều chuyển, đổi chức vụ, cho nghỉ việc |
| Kế toán / ban giám đốc | Cập nhật mức hoa hồng, xem và xuất bảng hoa hồng PT |
| Hệ thống | Kiểm tra ràng buộc, đồng bộ dữ liệu hiện tại mỗi ngày |

| Bảng | Nội dung chính |
|---|---|
| `CHI_NHANH` | Mã, tên, địa chỉ, SĐT, giờ mở/đóng cửa, sức chứa |
| `NHAN_VIEN` | Mã, họ tên, SĐT, chức vụ hiện tại, chi nhánh hiện tại (`MaCNHienTai`), lương cơ bản |
| `LICH_SU_CONG_TAC` | Nhân viên, chi nhánh, chức vụ trong giai đoạn, ngày bắt đầu, ngày kết thúc |
| `HOA_HONG_CHI_NHANH` | Chi nhánh, mức hoa hồng mỗi buổi, ngày bắt đầu, ngày kết thúc |
| `LICH_HEN_PT` (C) | PT, chi nhánh, ngày tập, giờ bắt đầu, trạng thái — A chỉ đọc |

## 3. Quy tắc nghiệp vụ chung

1. Hệ thống có cố định 12 chi nhánh. Chi nhánh chỉ được thêm và sửa thông tin, không xóa, không có trạng thái ngừng hoạt động.
2. Dịch vụ (gói Gym, Yoga, PT, Spa, vé lượt) có ở mọi chi nhánh. Sản phẩm bán tại chi nhánh nào thể hiện qua `TON_KHO` của Module D. Vì vậy không có bảng tiện ích riêng.
3. Ngày trong `LICH_SU_CONG_TAC` và `HOA_HONG_CHI_NHANH` là kiểu DATE, **tính cả ngày bắt đầu và ngày kết thúc**. `NgayKetThuc` rỗng nghĩa là chưa xác định ngày kết thúc. `NgayKetThuc` nếu có phải `>= NgayBatDau`.
4. Các giai đoạn công tác của cùng một nhân viên không chồng nhau, nên tại một ngày nhân viên thuộc tối đa một chi nhánh với một chức vụ.
5. Mọi thay đổi vị trí (chi nhánh, chức vụ, nghỉ việc) gửi trong ngày D có hiệu lực từ **D+1** theo giờ Asia/Saigon: giai đoạn cũ kết thúc hết ngày D, giai đoạn mới bắt đầu D+1. Không đổi giữa ngày, không hồi tố.
6. `NHAN_VIEN.MaCNHienTai` và `NHAN_VIEN.ChucVu` là bản lưu sẵn để đọc nhanh, luôn lấy từ giai đoạn công tác bao phủ hôm nay. Không có giai đoạn nào bao phủ hôm nay thì `MaCNHienTai` rỗng, nghĩa là **nhân viên đã nghỉ việc**.
7. Không xóa lịch sử công tác và lịch sử mức hoa hồng.
8. Các khoảng mức hoa hồng của cùng một chi nhánh không chồng nhau. Mức mới chỉ có hiệu lực **từ ngày mai**.
9. Một buổi được tính hoa hồng khi lịch PT có trạng thái **Đã hoàn thành** (gồm cả buổi hội viên không đến nhưng không hủy đúng hạn, theo quy trình của C). Buổi được tính cho chi nhánh mà PT thuộc về tại `NgayTap`, với mức hoa hồng của chi nhánh đó tại `NgayTap`.
10. Lịch PT của tháng đã qua không được sửa (C bảo đảm). Cộng với quy tắc 7–8, hoa hồng của tháng cũ tính lại lúc nào cũng ra cùng kết quả, nên không cần lưu kết quả chốt kỳ.
11. PT chỉ dạy tại chi nhánh đang công tác. Đổi vị trí bị từ chối nếu làm lịch PT đã đặt trở nên không hợp lệ (QT-A03).

## 4. QT-A01 — Cập nhật thông tin chi nhánh

**Ai làm:** quản trị hệ thống.

1. Chọn chi nhánh cần sửa, hoặc thêm chi nhánh mới khi dựng hệ thống.
2. Nhập tên, địa chỉ, SĐT, giờ mở cửa, giờ đóng cửa, sức chứa.
3. Hệ thống kiểm tra: dữ liệu bắt buộc đủ, mã chưa tồn tại (khi thêm), `GioMoCua < GioDongCua` trong cùng ngày, `SucChua` nguyên dương.
4. Lưu thay đổi.

**Ngoại lệ:** trùng mã; giờ đóng không sau giờ mở; sức chứa không nguyên dương → yêu cầu nhập lại. Yêu cầu xóa chi nhánh → từ chối.

**Ghi chú:** sức chứa chỉ dùng cho báo cáo tỷ lệ lấp đầy, không chặn lượt vào. Giờ hoạt động và sức chứa coi như cố định trong kỳ dữ liệu mô phỏng; hoạt động qua đêm nằm ngoài phạm vi.

## 5. QT-A02 — Tiếp nhận nhân viên

**Ai làm:** quản lý nhân sự.

1. Nhập họ tên, SĐT, chức vụ (PT, Lễ tân, Quản lý chi nhánh, Nhân viên bán hàng), lương cơ bản, chi nhánh tiếp nhận.
2. Hệ thống kiểm tra dữ liệu bắt buộc, chức vụ đúng danh mục, lương không âm, chi nhánh tồn tại.
3. Trong một giao dịch: tạo `NHAN_VIEN` với `ChucVu` và `MaCNHienTai` như đã nhập; tạo `LICH_SU_CONG_TAC` đầu tiên với `NgayBatDau` = hôm nay, `NgayKetThuc` rỗng, cùng chi nhánh và chức vụ.
4. Báo tiếp nhận thành công.

**Ngoại lệ:** thiếu hoặc sai dữ liệu → nhập lại. Tạo lịch sử lỗi → hoàn tác cả hồ sơ, không để nhân viên không có nơi công tác. Tiếp nhận với ngày bắt đầu trong tương lai nằm ngoài phạm vi.

## 6. QT-A03 — Điều chuyển nhân viên sang chi nhánh khác

**Ai làm:** quản lý nhân sự.

**Điều kiện trước:** nhân viên đang làm việc (có đúng một giai đoạn bao phủ hôm nay), chưa có thay đổi nào đang chờ hiệu lực, chi nhánh đích khác chi nhánh hiện tại.

1. Chọn nhân viên, chi nhánh đích; có thể đổi kèm chức vụ, nếu không thì giữ chức vụ cũ.
2. Hệ thống lấy ngày hôm nay D; ngày hiệu lực là D+1.
3. **Kiểm tra lịch PT vướng:** lấy các lịch của nhân viên đang ở trạng thái **Đã đặt** có `NgayTap >= D+1`. Lịch nào không còn hợp lệ với vị trí mới (không còn là PT, hoặc khác chi nhánh mới) thì từ chối toàn bộ yêu cầu và trả danh sách lịch vướng (mã lịch, ngày giờ, chi nhánh, lý do).
4. Nếu không vướng, trong một giao dịch: đặt `NgayKetThuc` của giai đoạn hiện tại = D; tạo giai đoạn mới với `NgayBatDau` = D+1, `NgayKetThuc` rỗng, chi nhánh và chức vụ mới.
5. `NHAN_VIEN` giữ nguyên trong ngày D; sang D+1 được QT-A05 đồng bộ.

**Ngoại lệ:**
- Chi nhánh đích trùng chi nhánh hiện tại → không dùng luồng này; chỉ đổi chức vụ thì dùng QT-A04.
- Không có hoặc có nhiều giai đoạn bao phủ hôm nay, hoặc đã có giai đoạn chờ bắt đầu ngày mai → từ chối.
- Còn lịch vướng → từ chối, giữ nguyên dữ liệu. Hệ thống **không** tự hủy, đổi PT hay đổi chi nhánh của lịch. Muốn điều chuyển thì các lịch đó phải được hội viên hủy đúng hạn hoặc diễn ra xong trước, rồi gửi lại yêu cầu.
- Hai yêu cầu đồng thời cho cùng nhân viên, hoặc đặt lịch PT đồng thời với điều chuyển → chỉ một bên thành công; bên sau kiểm tra lại trên dữ liệu mới nhất.
- Lỗi ở bất kỳ bước ghi nào → hoàn tác toàn bộ.

**Ví dụ:** bấm điều chuyển ngày 15/11 thì vị trí cũ kết thúc hết 15/11, vị trí mới bắt đầu 16/11.

## 7. QT-A04 — Cập nhật thông tin, lương và chức vụ

**Ai làm:** quản lý nhân sự.

1. Tìm nhân viên theo mã, tên hoặc chi nhánh hiện tại.
2. **Sửa SĐT, họ tên, lương cơ bản:** kiểm tra định dạng, lương không âm, rồi lưu thẳng vào `NHAN_VIEN`. Lương chỉ lưu giá trị hiện tại; lương quá khứ nằm ngoài phạm vi.
3. **Đổi chức vụ trong cùng chi nhánh:** làm như QT-A03 (bước 2–5) nhưng giữ chi nhánh. Ví dụ PT chuyển thành Lễ tân thì mọi lịch Đã đặt từ D+1 trở đi đều là lịch vướng.

**Ngoại lệ:** nhân viên đã nghỉ việc → không sửa chức vụ hay vị trí. Lương âm → từ chối. Chức vụ mới trùng chức vụ hiện tại → từ chối.

## 8. QT-A05 — Đồng bộ chi nhánh và chức vụ hiện tại mỗi ngày

**Ai làm:** job hệ thống, chạy đầu mỗi ngày (00:00 Asia/Saigon).

1. Với mỗi nhân viên, tìm giai đoạn công tác bao phủ hôm nay.
2. Có đúng một giai đoạn → cập nhật `NHAN_VIEN.MaCNHienTai` và `ChucVu` theo giai đoạn đó.
3. Không có giai đoạn nào → đặt `MaCNHienTai` rỗng (đã nghỉ việc), giữ `ChucVu` cuối cùng để tra cứu.

**Ngoại lệ:** có nhiều giai đoạn cùng bao phủ → báo dữ liệu bất thường, không tự chọn. Job chạy lại cho cùng kết quả. Nếu job chạy trễ, các chức năng cần vị trí chính xác tại một ngày (đặt lịch PT, tính hoa hồng) luôn đọc `LICH_SU_CONG_TAC`, không dựa vào bản lưu sẵn.

## 9. QT-A06 — Tra cứu lịch sử công tác

**Ai làm:** quản lý nhân sự, quản lý chi nhánh, kế toán.

1. Nhập nhân viên, có thể nhập thêm một ngày cần tra.
2. Có ngày: tìm giai đoạn thỏa `NgayBatDau <= Ngay AND (NgayKetThuc IS NULL OR Ngay <= NgayKetThuc)`, trả chi nhánh và chức vụ tại ngày đó.
3. Không có ngày: trả toàn bộ lịch sử theo thứ tự thời gian.

**Ngoại lệ:** không có giai đoạn nào → báo "không công tác tại ngày này". Nhiều giai đoạn → báo dữ liệu chồng lấn.

## 10. QT-A07 — Cập nhật mức hoa hồng của chi nhánh

**Ai làm:** kế toán / ban giám đốc.

1. Chọn chi nhánh, nhập mức hoa hồng mới mỗi buổi (số tiền không âm).
2. Hệ thống lấy hôm nay D. Trong một giao dịch: đặt `NgayKetThuc` = D cho khoảng mức đang hiệu lực; tạo khoảng mới với `NgayBatDau` = D+1, `NgayKetThuc` rỗng.
3. Khi dựng hệ thống, mỗi chi nhánh được tạo một khoảng mức đầu tiên.

**Ngoại lệ:** mức âm → từ chối. Đã có khoảng mới chờ bắt đầu ngày mai → từ chối (sửa lần nữa vào ngày khác). Không được tạo khoảng bắt đầu trong quá khứ.

## 11. QT-A08 — Tính và xem hoa hồng PT theo tháng

**Ai làm:** kế toán / quản lý chi nhánh (chỉ xem chi nhánh mình).

1. Chọn tháng, có thể lọc theo chi nhánh hoặc PT.
2. Hệ thống lấy các lịch PT có trạng thái **Đã hoàn thành** và `NgayTap` thuộc tháng.
3. Với từng buổi: tìm giai đoạn công tác của PT bao phủ `NgayTap` (phải có chức vụ PT và chi nhánh trùng `MaCN` của lịch); tìm mức hoa hồng của chi nhánh đó bao phủ `NgayTap`.
4. Nhóm theo PT và chi nhánh: số buổi, mức hoa hồng, thành tiền. PT điều chuyển giữa tháng sẽ có hai dòng, mỗi chi nhánh một dòng.
5. Hiển thị kèm danh sách buổi bất thường, nếu có.

**Ngoại lệ:**
- Buổi không tìm được giai đoạn công tác hợp lệ, hoặc chi nhánh của lịch khác chi nhánh trong lịch sử → đưa vào danh sách bất thường, không tính tự động.
- Không tìm được mức hoa hồng tại ngày đó → bất thường.
- Tháng chưa kết thúc, hoặc còn lịch Đã đặt đã qua giờ mà job của C chưa xử lý → kết quả ghi là **tạm tính**.
- Không có buổi nào → trả bảng rỗng.

## 12. QT-A09 — Cho nhân viên nghỉ việc

**Ai làm:** quản lý nhân sự.

**Điều kiện trước:** nhân viên đang làm việc, chưa có thay đổi nào đang chờ hiệu lực.

1. Chọn nhân viên, xác nhận nghỉ việc trong ngày D; ngày làm việc cuối là D.
2. Kiểm tra lịch PT vướng: còn lịch **Đã đặt** có `NgayTap >= D+1` thì từ chối và trả danh sách lịch.
3. Không vướng: đặt `NgayKetThuc` của giai đoạn hiện tại = D.
4. Sang D+1, QT-A05 đặt `MaCNHienTai` rỗng. Từ đó nhân viên không được chọn cho lịch PT hay hóa đơn mới.

**Ngoại lệ:** đã nghỉ việc, hoặc đang có thay đổi chờ hiệu lực → từ chối. Lỗi ghi → hoàn tác. Lịch sử cũ được giữ để tra cứu và tính hoa hồng các tháng trước.

## 13. Quan hệ với module khác

| Module | A cung cấp | A sử dụng |
|---|---|---|
| B | `CHI_NHANH(MaCN)` cho chi nhánh đăng ký của hội viên | — |
| C | `CHI_NHANH(MaCN, GioMoCua, GioDongCua)`; `NHAN_VIEN(MaNV)`; `LICH_SU_CONG_TAC(MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc)` để kiểm tra PT khi đặt lịch | `LICH_HEN_PT(MaLH, MaNV, MaCN, NgayTap, ThoiGianBatDau, TrangThai)` để tính hoa hồng và kiểm tra lịch vướng |
| D | `CHI_NHANH(MaCN, SucChua)`; `NHAN_VIEN(MaNV, MaCNHienTai)` để xác định chi nhánh lập hóa đơn | — |
