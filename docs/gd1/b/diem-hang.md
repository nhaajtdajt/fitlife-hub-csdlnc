# Điểm tích lũy và hạng hội viên — GD1-09

Chốt: xét hạng theo 12 tháng gần nhất, không reset theo năm/ngày đăng ký. Điểm tích lũy không hết hạn, chưa hỗ trợ đổi điểm. Đây là phần thiết kế B–D; thiết kế B đầy đủ đã hoàn thiện ở GD1-11.

## Dữ liệu

- HoiVien bổ sung TongChiTieu12T, DiemTichLuy, Hang: giá trị tổng hợp được lưu để đọc nhanh, ban đầu 0, 0, Silver.
- GhiNhanLoyalty(MaHD, DiemCong, ThoiDiemGhiNhan, DaLoaiKhoi12T). MaHD vừa là PK vừa FK tới HoaDon; một hóa đơn có tối đa một dòng, kể cả hóa đơn 0đ. DiemCong là phần nguyên TongTien / 100.000; DaLoaiKhoi12T ban đầu 0 nếu khoản chi còn trong cửa sổ, chỉ chuyển 0 → 1.
- Hội viên, số tiền và ngày mua lấy qua HoaDon(MaHD, MaHV, TongTien, NgayLap) bất biến, không sao chép vào lịch sử điểm. Không xóa dòng ghi nhận khi hóa đơn hết hạn xét hạng.
- MaHD xác định các thuộc tính của GhiNhanLoyalty; quan hệ này đạt BCNF theo PTH đó. Hang phụ thuộc TongChiTieu12T theo ngưỡng: phần lưu tổng/hạng của HoiVien là dư thừa có chủ đích, không được kết luận toàn quan hệ BCNF chỉ dựa trên MaHV. Phân tích đầy đủ tại thiet-ke-logic.md (GD1-11).

## Mốc 12 tháng

Quy ước thời gian: mỗi khoản có mốc hết hạn bằng NgayLap cộng 12 tháng theo Asia/Saigon, giữ giờ; nếu thiếu ngày tương ứng thì kẹp cuối tháng. Khoản được tính khi NgayLap <= T < MocHetHan. Ví dụ 15/11/2026 10:00 hết hạn xét hạng tại 15/11/2027 10:00; ngày 29/02/2028 chuyển thành 28/02/2029 cùng giờ. Không xóa hóa đơn hoặc trừ điểm ở mốc này.

Các giá trị lưu là bộ nhớ tổng hợp tại lần đồng bộ gần nhất. Trước khi hiển thị/sử dụng hạng hoặc thanh toán, phải xử lý các khoản đã hết hạn tới thời điểm T; tác vụ nền chỉ giúp cập nhật sớm, không phải căn cứ duy nhất để coi hạng còn đúng.

## Giao dịch

1. Thanh toán D và cập nhật B nằm trong cùng giao dịch. Với hội viên MaHV, lấy thời điểm T và xử lý các dòng ghi nhận chưa loại nhưng có mốc hết hạn <= T: giảm TongChiTieu12T bằng TongTien của chính hóa đơn đó, bật DaLoaiKhoi12T cùng giao dịch. Không giảm DiemTichLuy.
2. Kiểm tra MaHD và chủ hóa đơn/số tiền/ngày lập từ dữ liệu D đã ghi trong giao dịch, không tin giá trị khác từ lời gọi. Nếu đã có GhiNhanLoyalty thì không cộng lại tiền/điểm. Nếu chưa có: tạo dòng, cộng DiemCong; chỉ cộng TongTien vào tổng xét hạng nếu còn trong cửa sổ tại T. Khoản đã hết hạn khi xử lý bù được ghi cờ 1 và chỉ cộng điểm.
3. Tính Hang từ tổng sau đồng bộ: Silver < 20 triệu; Gold từ 20 đến dưới 50 triệu; Platinum từ 50 triệu. Đây là ngưỡng nhóm đang dùng, vẫn cần xác nhận với giảng viên theo #4. Việc mua mới tăng tổng sau đồng bộ; hạng so với giá trị lưu cũ có thể giảm nếu đồng thời vừa loại khoản quá hạn.
4. Mọi bước lỗi hoàn tác cả hóa đơn, ghi nhận và tổng/điểm/hạng; không cho số dư âm. Thanh toán, tác vụ nền và đọc hạng có đồng bộ phải giữ kết quả nhất quán; mỗi MaHD chỉ được ghi nhận một lần.

Tác vụ mỗi đêm chọn khoản có DaLoaiKhoi12T = 0 đã hết hạn, xử lý theo từng hội viên bằng cùng giao dịch trên rồi cập nhật hạng. Chạy bù lấy tất cả khoản đến hạn còn chưa loại, không chỉ hóa đơn hết hạn hôm trước; chạy lại không trừ lần hai. Thanh toán đồng thời với tác vụ không được cộng/trừ trùng hoặc ghi đè kết quả đồng bộ. Không quét/cộng toàn bộ hóa đơn 12 tháng trên đường thanh toán; chỉ xử lý phần đến hạn chưa được loại và hóa đơn mới. Chỉ mục phục vụ truy tìm khoản đến hạn/ghi nhận theo hội viên và giới hạn số học sẽ thiết kế, đo ở GĐ2.

## Ràng buộc và ảnh hưởng

| Ràng buộc | Thao tác cần kiểm tra |
|---|---|
| Một ghi nhận cho mỗi hóa đơn đã thanh toán; đúng chủ hóa đơn | Thêm/sửa MaHD; sửa/xóa HoaDon hoặc ghi nhận (bị cấm sau thanh toán) |
| DiemCong = phần nguyên TongTien / 100.000, điểm tổng không âm | Thêm/sửa/xóa ghi nhận; sửa TongTien; sửa DiemTichLuy |
| Tổng xét hạng bằng tổng tiền các ghi nhận chưa loại tại thời điểm đồng bộ | Thêm/xóa/sửa ghi nhận/cờ; sửa tổng hoặc hóa đơn; thời gian đến mốc hết hạn kích hoạt đồng bộ |
| Hạng khớp tổng sau đồng bộ, điểm không bị trừ do hết hạn | Sửa tổng/hạng/điểm; thao tác hết hạn phải giữ nguyên điểm |

Đối soát có thể tính lại tổng và điểm từ hóa đơn + ghi nhận để phát hiện sai lệch, nhưng không dùng cách cộng toàn lịch sử cho mỗi lần thanh toán. Dữ liệu triển khai mới phải khởi tạo đồng bộ; dữ liệu cũ cần đối soát trước khi bật cách lưu tổng này.

**Ánh xạ GD1-11:** GoiHoiVien cơ sở lưu MaGHV, MaHD, STT, DonVi, ngày và số dư; MaHV/MaGoi trong giao diện đọc trên đây lấy từ quan hệ dẫn xuất vGoiHoiVien qua hóa đơn/dòng, không phải FK/cột lưu lặp trên gói. Lược đồ cơ sở và ERD hiện hành tại [thiết kế B](thiet-ke-logic.md) thay thế phần minh họa gói trước GD1-11; giữ nguyên quy tắc PT/điểm.
