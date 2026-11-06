# Module D — Mô tả quy trình nghiệp vụ

Phạm vi: sản phẩm bán lẻ, tồn kho theo chi nhánh, hóa đơn và báo cáo. Dùng các cột đã công bố của A (`MaCN`, `SucChua`, `MaNV`), B (`MaHV`, `MaGoi`, `LoaiGoi`, `Gia`) và C (`MaCN`, `TGVao`, `TGRa`). Hóa đơn công bố `MaHD`, `MaHV`, `NgayLap`, `TongTien` cho B theo [hợp đồng giao diện](../../phan-cong.md).

Các số lượng trong tài liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Quyết định #3, #5 và #16 đã được chốt; quyết định #15 về tỷ lệ lấp đầy còn cần xác nhận cách xử lý lượt chưa quét ra.

## Tác nhân và dữ liệu

| Tác nhân | Trách nhiệm |
|---|---|
| Nhân viên bán hàng/lễ tân | Lập hóa đơn, chọn gói và sản phẩm, nhận thanh toán |
| Quản lý chi nhánh | Xem doanh thu, sản phẩm bán chạy và tồn kho tại chi nhánh |
| Ban giám đốc/kế toán | Xem doanh thu toàn hệ thống, đối chiếu hóa đơn và báo cáo |
| Hệ thống | Tính tiền, kiểm tra tồn, ghi hóa đơn/dòng hàng, cập nhật tồn và gọi thủ tục B cộng điểm/hạng |

| Dữ liệu | Nội dung |
|---|---|
| `SanPham` | Mã, tên, loại, giá bán hiện hành, trạng thái kinh doanh |
| `TonKho` | Chi nhánh, sản phẩm, số lượng tồn |
| `HoaDon` | Mã, hội viên, nhân viên lập, chi nhánh, thời điểm, tiền hàng, giảm giá, tổng sau giảm, phương thức thanh toán |
| `ChiTietHoaDon` | Hóa đơn, số thứ tự dòng, loại dòng, mã gói hoặc mã sản phẩm, số lượng, đơn giá chụp tại lúc bán, thành tiền |
| `CheckIn` (C) | Lượt vào/ra theo hội viên, chi nhánh và thời gian; dữ liệu nguồn cho báo cáo lấp đầy |
| `GoiDichVu` (B) | Mã gói, loại gói và giá hiện hành; dòng hóa đơn giữ giá bán riêng |

## Quy tắc nghiệp vụ chung

1. Một hóa đơn thuộc đúng một chi nhánh; nhân viên lập hóa đơn và hội viên phải tồn tại.
2. Hóa đơn có ít nhất một dòng hàng. Mỗi dòng là **một trong hai loại**: gói dịch vụ hoặc sản phẩm; không được đồng thời có cả `MaGoi` và `MaSP`, cũng không được bỏ trống cả hai.
3. Số lượng mỗi dòng là số nguyên dương. Gói dịch vụ bán theo đơn vị gói; sản phẩm có thể bán nhiều đơn vị.
4. Đơn giá dòng là giá tại thời điểm thanh toán. Giá danh mục thay đổi về sau không sửa lịch sử bán.
5. Tạm tính hóa đơn bằng tổng thành tiền các dòng; giảm giá không âm và không lớn hơn tạm tính; `TongTien = TamTinh - GiamGia`, không âm. `TongTien` đã gồm giảm giá và là giá trị truyền cho Module B.
6. Hình thức thanh toán thuộc `Tiền mặt`, `VNPay`, `Thẻ`. Chỉ thanh toán thành công mới ghi nhận hóa đơn bán hàng.
7. Với dòng sản phẩm, kho của chi nhánh trên hóa đơn phải đủ hàng. Ghi hóa đơn, dòng chi tiết và trừ kho là một giao dịch nguyên tử; lỗi ở bất kỳ bước nào thì hủy toàn bộ.
8. Với dòng gói, D ghi nhận nội dung và giá bán; sau thanh toán, Module B tạo `GoiHoiVien`. Việc tạo gói và cộng điểm/hạng phải nằm trong luồng giao dịch đã thống nhất giữa B và D.
9. Hóa đơn đã thanh toán không được sửa, xóa hoặc hoàn tiền theo quyết định #16. Sai sót trước khi thanh toán phải được sửa trước khi xác nhận giao dịch.
10. Không xóa vật lý sản phẩm đã xuất hiện trên hóa đơn hoặc tồn kho; chuyển trạng thái ngừng bán. Hóa đơn và dòng hóa đơn là dữ liệu lịch sử.
11. Báo cáo doanh thu dùng `NgayLap`, `MaCN`, `TongTien`; báo cáo sản phẩm bán chạy dùng số lượng trên dòng sản phẩm; báo cáo tồn kho đọc `TonKho`.
12. Tỷ lệ lấp đầy tại thời điểm T tạm tính bằng số lượt check-in tại chi nhánh có `TGVao <= T` và (`TGRa` rỗng hoặc `TGRa > T`) chia `SucChua`. Lượt còn mở lúc đóng cửa cần quy tắc tự đóng của C (#15).

## QT-D1. Quản lý danh mục sản phẩm

**Ai làm:** quản trị danh mục hoặc quản lý được phân quyền.

1. Người dùng tạo/sửa sản phẩm với mã, tên, loại, giá bán và trạng thái.
2. Hệ thống kiểm tra mã duy nhất, tên và loại bắt buộc, giá dương.
3. Khi bắt đầu kinh doanh sản phẩm tại chi nhánh, hệ thống khởi tạo dòng tồn kho cho cặp chi nhánh–sản phẩm với số lượng ban đầu được kiểm kê; nếu số lượng đầu kỳ khác 0 thì ghi một giao dịch kho mở đầu tương ứng.
4. Khi ngừng kinh doanh, hệ thống đánh dấu ngừng bán; không xóa sản phẩm đã có giao dịch.

**Lỗi:** mã trùng, giá không dương, dữ liệu thiếu; chi nhánh/sản phẩm đã được tham chiếu thì không xóa vật lý.

## QT-D2. Nhập hoặc điều chỉnh tồn kho

**Ai làm:** nhân viên kho/quản lý chi nhánh.

1. Chọn chi nhánh, sản phẩm và loại nghiệp vụ (nhập hàng, kiểm kê điều chỉnh).
2. Hệ thống xác nhận cặp sản phẩm–chi nhánh đã được mở kinh doanh và số lượng điều chỉnh hợp lệ.
3. Hệ thống cộng/trừ số lượng tồn và ghi nhận kết quả cùng giao dịch.
4. Người dùng xem số lượng mới.

**Lỗi:** cặp tồn kho không tồn tại, số lượng sau điều chỉnh âm hoặc đầu vào không phải số nguyên; không ghi một phần.

## QT-D3. Lập và thanh toán hóa đơn

**Ai làm:** nhân viên bán hàng/lễ tân tại chi nhánh.

1. Nhân viên chọn chi nhánh và tìm hội viên theo mã. Hệ thống ghi nhận nhân viên lập hóa đơn.
2. Nhân viên thêm một hay nhiều dòng sản phẩm và/hoặc gói dịch vụ. Với mỗi dòng, hệ thống kiểm tra mặt hàng đang bán, số lượng và chụp đơn giá hiện hành.
3. Hệ thống tính thành tiền từng dòng, tổng tiền hàng, giảm giá và số phải trả. Nhân viên xác nhận phương thức thanh toán.
4. Trước khi chấp nhận thanh toán, hệ thống kiểm tra tồn kho của mọi dòng sản phẩm tại chi nhánh; không đủ hàng thì báo dòng thiếu và không trừ kho.
5. Khi thanh toán thành công, hệ thống trong một giao dịch: tạo `HoaDon`, tạo toàn bộ `ChiTietHoaDon`, trừ tồn kho theo số lượng, và gọi `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)` của B.
6. Module B xử lý quyền lợi gói và điểm/hạng theo phần việc của B. D chỉ hoàn tất nếu các bước tích hợp bắt buộc thành công.
7. Hệ thống trả mã hóa đơn và biên nhận cho nhân viên/hội viên.

**Lỗi:** thanh toán bị từ chối, sản phẩm ngừng bán, tồn kho thiếu, số tiền không hợp lệ, hoặc thủ tục B thất bại. Giao dịch phải rollback để không tạo hóa đơn thiếu dòng, không trừ kho một phần và không có trạng thái tiền/điểm lệch nhau. Yêu cầu về ranh giới giao dịch khi B tạo `GoiHoiVien` cần D và B xác nhận ở bước 2.3.

## QT-D4. Tra cứu và đối chiếu hóa đơn

**Ai làm:** nhân viên bán hàng, kế toán hoặc quản lý theo quyền.

1. Người dùng lọc theo mã hóa đơn, hội viên, khoảng ngày, chi nhánh hoặc nhân viên lập.
2. Hệ thống trả thông tin đầu hóa đơn và các dòng, gồm giá bán tại thời điểm giao dịch.
3. Kế toán đối chiếu tổng tiền đầu hóa đơn với các dòng và khoản giảm giá.

**Lỗi:** bộ lọc ngày không hợp lệ; không tìm thấy thì trả danh sách rỗng. Hóa đơn đã thanh toán chỉ xem/đối chiếu, không sửa.

## QT-D5. Báo cáo doanh thu, sản phẩm và tồn kho

**Ai làm:** quản lý chi nhánh hoặc ban giám đốc.

- Doanh thu: chọn khoảng thời gian và phạm vi chi nhánh; tổng hợp `TongTien` theo ngày/tháng/quý/năm và so sánh giữa chi nhánh.
- Sản phẩm bán chạy: cộng số lượng dòng sản phẩm trong kỳ, có thể lọc chi nhánh.
- Tồn kho: xem số lượng hiện có theo chi nhánh và sản phẩm.
- Gói dịch vụ doanh thu cao: cộng thành tiền các dòng gói trong 6 tháng gần nhất.
- Tỷ lệ lấp đầy: đọc `CheckIn` của C, chia số hội viên đang có mặt theo sức chứa chi nhánh tại các khung giờ.

Báo cáo không sửa dữ liệu giao dịch. Kỳ không có giao dịch trả kết quả 0/rỗng phù hợp. Báo cáo phải lọc khoảng thời gian rõ ràng để có thể đánh giá ở giai đoạn vật lý.

## Quan hệ với module khác

| Module | D cung cấp | D sử dụng |
|---|---|---|
| A — Chi nhánh & Nhân sự | Không sở hữu dữ liệu của A | `ChiNhanh(MaCN, SucChua)`, `NhanVien(MaNV)` |
| B — Hội viên & Gói | `HoaDon(MaHD, MaHV, NgayLap, TongTien)`; gọi thủ tục cộng điểm/hạng | `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi, Gia)` |
| C — Check-in & Lịch PT | Báo cáo đọc dữ liệu C, không ghi vào bảng C | `CheckIn(MaCN, TGVao, TGRa)` để tính lấp đầy |

