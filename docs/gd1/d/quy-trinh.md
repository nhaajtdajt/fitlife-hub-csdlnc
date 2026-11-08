# Module D — Mô tả quy trình nghiệp vụ

Phạm vi: sản phẩm bán lẻ, tồn kho theo chi nhánh, hóa đơn và báo cáo. Dùng các cột đã công bố của A (`MaCN`, `SucChua`, `GioMoCua`, `GioDongCua`, `MaNV`), B (`MaHV`, `MaGoi`, `LoaiGoi`, `Gia`) và C (`MaCN`, `TGVao`, `TGRa`, `SoHieuCuaRa`). Hóa đơn công bố `MaHD`, `MaHV`, `NgayLap`, `TongTien` cho B theo [hợp đồng giao diện](../../phan-cong.md).

Các số lượng trong tài liệu lấy từ [bảng giả định chung](../../quyet-dinh.md#bảng-giả-định-số-liệu-chung). Quyết định #3, #5 và #16 đã được chốt; GD1-07/#15 đã chốt chỉ báo cáo lấp đầy, không chặn cổng; quy ước tự đóng lượt và giới hạn dữ liệu mô phỏng được nêu bên dưới.

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
| `ChiTietHoaDon` | Hóa đơn, số thứ tự dòng, loại dòng, mã gói hoặc mã sản phẩm, số lượng, đơn giá chụp tại lúc bán; thành tiền tính khi đọc |
| `CheckIn` (C) | Lượt vào/ra theo hội viên, chi nhánh và thời gian; dữ liệu nguồn cho báo cáo lấp đầy |
| `GoiDichVu` (B) | Mã gói, loại gói và giá hiện hành; dòng hóa đơn giữ giá bán riêng |

## Quy tắc nghiệp vụ chung

1. Một hóa đơn thuộc đúng một chi nhánh; nhân viên lập hóa đơn và hội viên phải tồn tại.
2. Hóa đơn có ít nhất một dòng hàng. Mỗi dòng là **một trong hai loại**: gói dịch vụ hoặc sản phẩm; không được đồng thời có cả `MaGoi` và `MaSP`, cũng không được bỏ trống cả hai.
3. Số lượng mỗi dòng là số nguyên dương. Gói dịch vụ bán theo đơn vị gói; sản phẩm có thể bán nhiều đơn vị.
4. Đơn giá dòng là giá tại thời điểm thanh toán. Giá danh mục thay đổi về sau không sửa lịch sử bán. GD1-12: chỉ lưu SoLuong và DonGia; ThanhTien = SoLuong × DonGia là giá trị truy vấn/hiển thị, không ghi vào ChiTietHoaDon.
5. Tạm tính hóa đơn bằng tổng thành tiền các dòng; giảm giá không âm và không lớn hơn tạm tính; `TongTien = TamTinh - GiamGia`, không âm. `TongTien` đã gồm giảm giá và là giá trị truyền cho Module B.
6. Hình thức thanh toán thuộc `Tiền mặt`, `VNPay`, `Thẻ`. Chỉ thanh toán thành công mới ghi nhận hóa đơn bán hàng.
7. Với dòng sản phẩm, kho của chi nhánh trên hóa đơn phải đủ hàng. Ghi hóa đơn, dòng chi tiết và trừ kho là một giao dịch nguyên tử; lỗi ở bất kỳ bước nào thì hủy toàn bộ.
8. Với dòng gói, D ghi nhận nội dung và giá bán; sau thanh toán, Module B tạo `GoiHoiVien`. Việc tạo gói và cộng điểm/hạng phải nằm trong luồng giao dịch đã thống nhất giữa B và D.
9. Hóa đơn đã thanh toán không được sửa, xóa hoặc hoàn tiền theo quyết định #16. Sai sót trước khi thanh toán phải được sửa trước khi xác nhận giao dịch.
10. Không xóa vật lý sản phẩm đã xuất hiện trên hóa đơn hoặc tồn kho; chuyển trạng thái ngừng bán. Hóa đơn và dòng hóa đơn là dữ liệu lịch sử.
11. Mọi báo cáo doanh thu dùng số tiền sau giảm giá (#18): tổng hóa đơn dùng NgayLap/MaCN/TongTien; theo gói/sản phẩm dùng doanh thu dòng đã phân bổ giảm giá. Báo cáo sản phẩm bán chạy theo số lượng không thay đổi; tồn kho đọc TonKho.
12. Tỷ lệ lấp đầy tại thời điểm T = 100 × số lượt CheckIn cùng chi nhánh thỏa `TGVao <= T AND (TGRa IS NULL OR TGRa > T)` / SucChua. SucChua là số nguyên dương; tỷ lệ có thể vượt 100%, không chặn cổng hoặc ép kết quả về 100%. Lượt quên quét ra theo quy ước tự đóng C QT-C3 (#15).

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
2. Nhân viên thêm một hay nhiều dòng sản phẩm và/hoặc gói dịch vụ. Với mỗi dòng, hệ thống kiểm tra mặt hàng đang bán, số lượng và chụp đơn giá hiện hành. Nếu mua PT mà chưa có thẻ dịch vụ cấp quyền vào còn hiệu lực hoặc vé vào còn lượt, phải thêm thẻ/vé vào hóa đơn (có lựa chọn 1 tháng hoặc một lượt). B kiểm tra điều kiện này trước khi hoàn tất; vé mua kèm không được giữ riêng cho giờ PT.
3. Hệ thống tính thành tiền trước giảm từng dòng, tổng tiền hàng, giảm giá và số phải trả. Phân bổ giảm giá theo tỷ lệ toàn bộ các dòng và quy tắc làm tròn GD1-08 trong thiết kế logic; hiển thị giảm phân bổ và tiền sau giảm, kiểm tra tổng khớp hóa đơn trước thanh toán. Với Gym/Yoga theo hạn, B tính ngày bắt đầu/hết hạn nối tiếp sau gói cùng loại cuối cùng đã mua; D hiển thị rõ gói dùng ngay hay chờ kích hoạt để hội viên xác nhận trước khi thanh toán. Việc tính ngày và tạo gói phải được B kiểm tra nguyên tử khi ghi, tránh hai lần mua đồng thời chồng khoảng. Nhân viên xác nhận phương thức thanh toán.
4. Trước khi chấp nhận thanh toán, hệ thống kiểm tra tồn kho của mọi dòng sản phẩm tại chi nhánh; không đủ hàng thì báo dòng thiếu và không trừ kho.
5. Khi thanh toán thành công, hệ thống trong một giao dịch: tạo `HoaDon`, tạo toàn bộ `ChiTietHoaDon`, trừ tồn kho theo số lượng, và yêu cầu B cập nhật điểm/hạng theo hội viên, mã hóa đơn, tổng tiền và ngày lập.
6. Module B xử lý quyền lợi gói và điểm/hạng theo phần việc của B. Mỗi đơn vị gói PT trên hóa đơn tạo MaGHV mới, bắt đầu ngày mua, buổi/hạn riêng; không nhập số buổi vào gói cũ hoặc đổi lịch đã đặt. D chỉ hoàn tất nếu các bước tích hợp bắt buộc thành công.
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

- Doanh thu: chọn khoảng thời gian và phạm vi chi nhánh; tổng hợp TongTien sau giảm theo ngày/tháng/quý/năm, mỗi hóa đơn đúng một lần. Không cộng TongTien lặp sau khi nối sang nhiều dòng chi tiết. Tổng doanh thu tất cả gói cộng tất cả sản phẩm phải khớp tổng hóa đơn trong cùng phạm vi; riêng phần gói không cần bằng toàn hóa đơn có cả sản phẩm.
- Sản phẩm bán chạy: cộng số lượng dòng sản phẩm trong kỳ, có thể lọc chi nhánh.
- Tồn kho: xem số lượng hiện có theo chi nhánh và sản phẩm.
- Gói dịch vụ doanh thu cao: phân bổ giảm giá trên toàn bộ dòng của từng hóa đơn trong 6 tháng gần nhất, sau đó mới lọc dòng gói, nhóm theo MaGoi và cộng DoanhThuDong sau giảm. Sản phẩm tính doanh thu theo cùng quy tắc; không dùng ThanhTien trước giảm để xếp hạng doanh thu.
- Tỷ lệ lấp đầy: đọc CheckIn của C theo công thức tại T. Báo cáo theo giờ lấy ảnh chụp tại đầu từng khung (ví dụ 17:00, 18:00), hiển thị rõ thời điểm; không lấy tổng lượt vào trong giờ làm số người đồng thời và không gọi ảnh chụp là trung bình cả giờ.

Vào đúng T được tính; ra đúng T không còn được tính. Một hội viên có tối đa một phiên mở nên đếm phiên hợp lệ tương ứng số người. Không có người thì trả 0% khi sức chứa hợp lệ; sức chứa thiếu/0/âm thì báo không tính được do dữ liệu cấu hình, không chia cho 0 hoặc trả 0% giả.

Quy ước mô phỏng: giờ hoạt động cùng ngày Asia/Saigon, giờ mở < giờ đóng, giờ hoạt động/sức chứa cố định trong kỳ dữ liệu. C ghi giờ ra ước tính tại giờ đóng cửa cho lượt quên ra và giữ SoHieuCuaRa rỗng. D hiển thị số lượt tự đóng trong kỳ để phân biệt chất lượng dữ liệu. Nếu còn lượt chưa đóng dù mốc đóng cửa của ngày vào đã qua, báo cáo là tạm tính và tính lại sau khi C chạy bù; không âm thầm coi là báo cáo kỳ đã hoàn tất. Không suy ra khách thực sự ở lại đến giờ đóng cửa. Thay đổi sức chứa theo lịch sử hoặc hoạt động qua đêm/24 giờ cần thiết kế bổ sung, không nằm trong bộ giả định này.

Báo cáo không sửa dữ liệu giao dịch. Kỳ không có giao dịch trả kết quả 0/rỗng phù hợp. Báo cáo phải lọc khoảng thời gian rõ ràng để có thể đánh giá ở giai đoạn vật lý.

## Quan hệ với module khác

| Module | D cung cấp | D sử dụng |
|---|---|---|
| A — Chi nhánh & Nhân sự | Không sở hữu dữ liệu của A | `ChiNhanh(MaCN, SucChua, GioMoCua, GioDongCua)`, `NhanVien(MaNV)` |
| B — Hội viên & Gói | `HoaDon(MaHD, MaHV, NgayLap, TongTien)`; gọi thủ tục cộng điểm/hạng | `HoiVien(MaHV)`, `GoiDichVu(MaGoi, LoaiGoi, Gia)` |
| C — Check-in & Lịch PT | Báo cáo đọc dữ liệu C, không ghi vào bảng C | `CheckIn(MaCN, TGVao, TGRa, SoHieuCuaRa)` để tính lấp đầy |


GD1-09: D gọi B trong cùng giao dịch thanh toán; B kiểm tra đầu vào với HoaDon, ghi nhận duy nhất theo MaHD và cập nhật tổng/điểm/hạng nguyên tử. Hết 12 tháng chỉ loại tiền khỏi tổng xét hạng, giữ điểm. Hợp đồng tại [thiết kế điểm/hạng B](../b/diem-hang.md).

**GD1-11 — tạo gói từ dòng hóa đơn:** mỗi GoiHoiVien lưu (MaHD,STT,DonVi), FK dòng và UNIQUE bộ ba; dòng gói số lượng n tạo đúng n gói, dòng sản phẩm tạo 0. Xử lý lại cùng MaHD trả gói đã có; không tạo lại gói/cộng điểm/trừ kho. Hóa đơn/gói/điểm/kho phải cùng được ghi nhận đầy đủ hoặc cùng hoàn tác khi lỗi. Thông tin chủ và danh mục đọc qua vGoiHoiVien. Chi tiết tại [thiết kế B](../b/thiet-ke-logic.md).
