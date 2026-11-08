# Rà soát cuối toàn bộ GĐ1 — 08/11/2026

## Kết luận

Hồ sơ đã có đủ nhóm đầu ra chính nhưng **chưa nên coi là bản cuối sẵn sàng nộp**. Kiểm tra cấu trúc đạt không chứng minh quy trình và ràng buộc đã đầy đủ. Lần rà này ghi 8 nhóm việc cần sửa/làm rõ, không tự thay quyết định nghiệp vụ và không triển khai GĐ2.

Nguồn đối chiếu: PDF gốc `2627-CSDLNC-DA.CLC.pdf` đã đọc văn bản cả 4 trang; [bản đề](../../de.md), [yêu cầu từng giai đoạn](../../required.md), quyết định người dùng trong [quyết định chung](../quyet-dinh.md). Các bổ sung đã được người dùng chốt (PT độc lập cổng, vé lượt, gia hạn nối tiếp, giờ PT tròn…) là thiết kế được phép bổ sung, không coi là lỗi chỉ vì đề không ghi chi tiết.

## Ma trận đầu ra GĐ1

| Yêu cầu | A | B | C | D / bản tổng |
|---|---|---|---|---|
| Quy trình chi tiết | Có 6 quy trình; thiếu biên tiếp nhận/nghỉ việc, cập nhật tiện ích theo khoảng ngày | Có 7 quy trình và hai tài liệu PT/điểm | Có 6 quy trình; thiếu mô tả đầy đủ một số chức năng đã kê trong tần suất | Có 5 quy trình; cần nói rõ ghi nhật ký kho trong các bước |
| Tên chức năng, mô tả, tần suất, mức | 12 dòng; chốt hoa hồng chưa thể hiện ghi, mô tả đồng bộ chi nhánh không khớp bảng | 15 dòng; hiện trạng cũ có nhãn nhưng cần tách rõ khỏi thiết kế mới | 14 dòng | 11 dòng; đủ 52 nhãn toàn nhóm |
| ERD và ràng buộc ngoài hình | Có bản hiện hành | Có bản hiện hành, nhưng sơ đồ phụ PT vẫn dùng cột cũ | Có; minimum 1 cửa/chi nhánh chưa khớp luồng tạo chi nhánh | Có; bản tổng 20 thực thể |
| Lược đồ, khóa, PTH, dạng chuẩn | Có | Có, ngoại lệ HoiVien 2NF đã nêu | Có | Có; 33 FK tổng khớp cột, 5 FK ghép |
| Thuộc tính và miền | Có | Có | Có | Có |
| Bối cảnh/nội dung/bảng ảnh hưởng | Có nhưng cần bổ sung biên nghiệp vụ | Có | Một số miền chỉ nằm trong từ điển/business rules | Một số miền SanPham chưa có bảng ảnh hưởng; 14 ràng buộc tổng không thay thế các miền còn thiếu |

## Bảng nhóm theo dõi và góp ý

Bàn giao ngày 08/11/2026; tạm dừng công việc tại đây theo yêu cầu của nhóm trưởng. Các hướng sửa trong báo cáo là đề xuất để nhóm xem xét, không phải quyết định nghiệp vụ mới. Mỗi người đọc phần module mình và góp ý cả các điểm liên module; nhóm có thể đưa phương án khác nếu nêu căn cứ từ đề và ảnh hưởng tới các tài liệu liên quan. Chưa bắt đầu triển khai GĐ2.

| Mã | Vấn đề cần xử lý | Module phối hợp | Bước tiếp theo / điều kiện hoàn tất | Trạng thái | Đề xuất / phản hồi của nhóm | Bản sửa / người kiểm tra |
|---|---|---|---|---|---|---|
| LC-01 | Thiếu quy trình cho chức năng đã kê | A, C | Bổ sung luồng tiện ích theo ngày, quản lý cửa, tra cứu và hiệu suất HLV; đối chiếu bảng tần suất | Chờ nhóm xem xét | Chưa có — A/C ghi đề xuất | Chưa có |
| LC-02 | Ngày tiếp nhận và nghỉ việc chưa khép kín | A, C | Thống nhất ngày hiệu lực/trạng thái nhân viên và lịch PT vướng; cập nhật quy trình, ràng buộc | Chờ nhóm thống nhất nghiệp vụ | Chưa có — không tự đổi lịch hoặc xóa lịch sử | Chưa có |
| LC-03 | Chi nhánh bắt buộc có cửa nhưng luồng tạo chưa bảo đảm | A, C | Chọn rõ cardinality khi cấu hình/hoạt động hoặc tạo cửa cùng chi nhánh; đồng bộ ERD và quy trình | Chờ nhóm thống nhất mô hình | Chưa có — đọc hai phương án trong LC-03 | Chưa có |
| LC-04 | Thiếu bảng ảnh hưởng một số ràng buộc miền | C, D | Bổ sung bối cảnh, nội dung và Thêm/Xóa/Sửa cho miền cửa và sản phẩm | Chờ nhóm xem xét | Chưa có — C/D đối chiếu từ điển thuộc tính | Chưa có |
| LC-05 | Tần suất chưa khớp thao tác chốt/đồng bộ | A; B đối chiếu phần hiện trạng | Nêu đúng đọc+ghi khi chốt; chi nhánh hiện tại suy ra từ lịch sử; tách hiện trạng khỏi thiết kế chọn | Chờ nhóm xem xét | Chưa có — giữ cách đếm nghiệp vụ PT đã chốt | Chưa có |
| LC-06 | Còn sơ đồ phụ/câu chờ chốt cũ | A, B, D | Đồng bộ hình PT với gói gắn dòng hóa đơn và thay câu cũ bằng quyết định hiện hành | Chờ nhóm xem xét | Chưa có — không đổi chính sách đã chốt chỉ để khớp hình cũ | Chưa có |
| LC-07 | Quy trình kho chưa nêu rõ ghi nhật ký | D; B đối chiếu thanh toán | Nêu ghi GiaoDichKho cùng thay đổi tồn, liên kết hóa đơn/chi nhánh và lượng gộp sản phẩm | Chờ nhóm xem xét | Chưa có — chỉ bổ sung đặc tả, không viết SQL | Chưa có |
| LC-08 | Quét/đặt PT ngoài giờ và tự đóng phiên chưa thống nhất | A, C; D đối chiếu báo cáo | Chốt cách nhận/từ chối ngoài giờ hoặc xử lý ngoại lệ hợp lệ; giữ sức chứa chỉ phục vụ báo cáo | Chờ nhóm thống nhất nghiệp vụ | Chưa có — không tự mở rộng ca qua đêm | Chưa có |

Cách cập nhật:

1. Ghi đề xuất theo đúng mã LC, nêu tình huống ví dụ và tài liệu bị ảnh hưởng; có thể ghi trực tiếp vào cột phản hồi hoặc dẫn liên kết Issue/PR của nhóm. Không xóa nhận xét cũ khi chưa giải quyết.
2. Với LC-02/03/08, thống nhất phương án trong nhóm trước khi đổi quy tắc chung. Nếu có ý kiến khác về phát hiện, ghi căn cứ và kết luận vào bảng thay vì âm thầm bỏ mục.
3. Sau khi sửa, điền đường dẫn bản sửa và người kiểm tra; dùng trạng thái Chờ nhóm xem xét → Đã thống nhất → Đang sửa → Chờ kiểm tra → Đã kiểm tra. Chỉ đóng khi quy trình, tần suất, ERD, logic và bản tổng bị ảnh hưởng đã khớp nhau.
4. Giữ nguyên 17 mã GD1 gốc trong [sổ theo dõi](theo-doi-ra-soat.md); LC là danh sách phát hiện của lần rà cuối, không đánh lại số hay làm mất việc cũ. Mục không thuộc đề/phạm vi có thể ghi Không áp dụng kèm lý do đã được nhóm thống nhất.

Việc chuẩn bị bản nộp: bổ sung tên/MSSV, nội dung đánh giá cá nhân/nhóm, liên kết công cụ theo dõi và xác nhận ngưỡng hạng; không tự điền thay các thành viên. Phần phân tích chi tiết và căn cứ của từng LC nằm ngay bên dưới.

## Danh sách cần xử lý

### LC-01 — Chức năng đã kê nhưng quy trình chưa mô tả đủ (A/C)

**Mức: cần bổ sung theo required.md — quy trình nghiệp vụ chi tiết.**

- [Tần suất C](c/bang-tan-suat.md): C-F03 tra cứu người có mặt, C-F05 quản lý cửa, C-F11 lịch sử tập/lịch PT, C-F13 hiệu suất HLV đều đã nằm trong phạm vi. [Quy trình C](c/quy-trinh.md) hiện dừng ở QT-C1–C6, chưa có luồng tương ứng đủ đầu vào, kiểm tra, kết quả và trường hợp rỗng/sai dữ liệu.
- Ví dụ C-F13 cần nói số buổi lấy lịch hoàn thành, điểm trung bình lấy các đánh giá hiện có và hiển thị thế nào khi chưa có đánh giá. Đây là mô tả nghiệp vụ, không phải viết truy vấn GĐ2.
- A QT-A01 mới nói chọn danh sách tiện ích rồi lưu; trong khi CungCapTienIch có nhiều giai đoạn NgayBatDau/NgayKetThuc. Chưa mô tả ngừng/cung cấp lại tiện ích theo ngày mà không ghi đè lịch sử.
- Hướng sửa tối thiểu: bổ sung các luồng ngắn trong file hiện có, giữ nguyên bảng và chức năng đã chọn.

### LC-02 — Vòng đời nhân viên chưa khép kín (A/C)

**Mức: cần sửa logic.**

- [QT-A02](a/quy-trinh.md) cho nhập ngày bắt đầu nhưng không ràng buộc ngày này với hôm nay; phần kết quả lại khẳng định nhân viên có đúng một giai đoạn đang hiệu lực. [A-R06](a/thiet-ke-logic.md) cũng yêu cầu nhân viên đang làm có giai đoạn hôm nay. Ví dụ tạo hồ sơ đang làm ngày 08/11 nhưng bắt đầu 09/11 không thỏa kết quả đó.
- NhanVien có TrangThaiLamViec=đang làm/nghỉ việc và A-F05 có cập nhật trạng thái, nhưng QT-A04 chỉ nêu sửa liên hệ/lương/chức vụ. Chưa nói nghỉ việc đóng lịch sử vào ngày nào và xử lý lịch PT tương lai ra sao. Nếu chỉ đổi trạng thái mà lịch sử PT vẫn mở, điều kiện chọn PT theo lịch sử ở C vẫn có thể nhận lịch mới.
- Hướng sửa: chốt phạm vi tiếp nhận (chỉ ngày có hiệu lực ngay hay cho hồ sơ chờ); mô tả nghỉ việc và kiểm tra lịch vướng theo cùng nguyên tắc bảo toàn lịch sử. Không chỉ lọc trạng thái hiện tại để loại buổi dạy quá khứ khỏi hoa hồng.

### LC-03 — Một chi nhánh bắt buộc có cửa nhưng luồng tạo chưa bảo đảm (A/C/tổng)

**Mức: cần thống nhất mô hình với quy trình.**

- [BR-C01](c/business-rules.md), [ERD C](c/erd.md) và [ERD tổng](erd-tong.md) ghi ChiNhanh có 1..N cửa.
- QT-A01 tạo chi nhánh chỉ lưu thông tin và tiện ích, không yêu cầu tạo cửa cùng lúc; quản lý cửa là chức năng riêng của C. Có thể tạo chi nhánh không có cửa, trái minimum của ERD hiện tại.
- Chưa có ràng buộc/bảng ảnh hưởng riêng cho việc xóa hoặc chuyển cửa cuối cùng.
- Hướng sửa: chọn rõ 0..N cửa trong giai đoạn cấu hình và ít nhất một cửa khi đưa vào hoạt động, hoặc giữ 1..N và mô tả tạo cửa bắt buộc cùng chi nhánh. Đây là quyết định cardinality GĐ1, không cần cài SQL.

### LC-04 — Ràng buộc miền chưa được đưa đầy đủ vào bảng ảnh hưởng (C/D)

**Mức: thiếu đầu ra cụ thể của required.md.**

- [Từ điển D](d/thiet-ke-logic.md) nêu SanPham.TenSP bắt buộc, GiaBan>0; QT-D1 còn nêu LoaiSP bắt buộc. D-R01–D-R08 chủ yếu bao phủ hóa đơn/kho/sức chứa, chưa có hàng bối cảnh/nội dung/Thêm-Xóa-Sửa cho các miền SanPham này. T-R01–T-R14 cũng chưa bù phần đó.
- C có SoHieuCua nguyên dương, ChucNang thuộc Vào/Ra/Hai chiều trong từ điển/BR-C11, nhưng bảng C-R chưa nêu đầy đủ kiểm tra miền khi thêm/sửa chính CuaKiemSoat.
- Hướng sửa: bổ sung hàng ràng buộc miền tương ứng, nêu đúng cột sửa; không cần tạo thêm bảng nghiệp vụ hoặc thêm tính năng.

### LC-05 — Bảng tần suất A chưa phản ánh thao tác đã chốt (A/B)

**Mức: cần sửa mô tả, không phải thay toàn bộ số liệu.**

- [A-F10](a/bang-tan-suat.md) vẫn có loại Đọc nặng; phần giải thích lại gọi đây là 12 lần chốt kỳ/tháng. Chốt kỳ thực sự ghi KyHoaHong/ChiTietHoaHong theo [thiết kế A](a/thiet-ke-logic.md). Cần tách xem trước/chốt hoặc ghi rõ đọc+ghi cho lượt chốt; không tự tính hai lần cùng một nghiệp vụ.
- A-F12 và QT-A04 nói cập nhật cả chi nhánh hiện tại lưu trên nhân viên, trong khi lược đồ NhanVien không có MaCN và thiết kế A đã xác định chi nhánh hiện tại suy từ lịch sử. Cần thống nhất chỉ ChucVu được đồng bộ lưu, chi nhánh hiện tại là kết quả tra cứu.
- B CN-B10 đã ghi rõ là hiện trạng cũ và cuối file có thiết kế mới, nên không kết luận B chưa sửa nghiệp vụ. Tuy nhiên bảng danh sách chức năng dùng cho bản nộp nên mô tả thiết kế được chọn; hiện trạng cộng lại 12 tháng giữ ở đoạn phân tích vấn đề để người đọc không lẫn.
- Kiểm tra lại 52 nhãn theo ngưỡng: tất cả nhãn đang đúng với số giả định; không yêu cầu đổi cách đếm PT theo đợt đã chốt.

### LC-06 — Tài liệu phụ còn cấu trúc/quyết định cũ (A/B/D)

**Mức: cần dọn mâu thuẫn trình bày.**

- [Sơ đồ PT phụ B](b/goi-pt.md) vẫn vẽ GoiHoiVien.MaHV/MaGoi là FK trực tiếp. Cuối file có ghi bị thay thế bởi GD1-11, nhưng hình vẫn rất dễ bị lấy nhầm khi đọc/in riêng. Bản hiện hành và bản tổng đúng là GoiHoiVien liên kết (MaHD,STT), suy chủ/loại qua hóa đơn/dòng.
- A quy tắc chung số 11 còn ghi công thức hoa hồng là đề xuất cần chốt trước thiết kế, trong khi QD06/14 và thiết kế A đã chốt cơ chế.
- D QT-D3 còn ghi ranh giới tạo gói cần B/D xác nhận ở bước 2.3, trong khi nguyên tử hóa đơn/gói/điểm/kho đã chốt ở GD1-11. Chỉ chữ ký/cách cài thủ tục mới thuộc GĐ2.
- Một số chỗ trong A/B vẫn nhắc thao tác khóa cụ thể dù đã tách ghi chú GĐ2. Không phải lỗi chức năng, nhưng cần sửa thành yêu cầu nhất quán để hồ sơ GĐ1 gọn và thống nhất.
- Hướng sửa: cập nhật hoặc gỡ hình phụ lỗi thời; thay câu chờ chốt bằng dẫn chiếu quyết định hiện hành. Không đổi chính sách đã được người dùng duyệt.

### LC-07 — Luồng kho chưa nói rõ ghi nhật ký bắt buộc (D)

**Mức: thiếu bước trong mô tả quy trình.**

- [QT-D2](d/quy-trinh.md) nói cộng/trừ tồn và ghi nhận kết quả, QT-D3 nói trừ tồn khi bán, nhưng không chỉ rõ tạo GiaoDichKho và trường liên kết/lượng tương ứng ở các bước này.
- [D-R03](d/thiet-ke-logic.md) yêu cầu SoLuongTon bằng tổng nhật ký; mọi lần thay đổi phải có nhật ký. QT-D1 đã nói rõ nhật ký mở đầu, nhưng hai luồng thường xuyên chưa rõ bằng mức đó.
- Ví dụ tồn 10, bán 2: nếu làm đúng các bước được nêu tường minh là giảm TonKho còn 8 nhưng không tạo xuất −2, tổng nhật ký vẫn 10, vi phạm D-R03. Đây là thiếu diễn đạt quy trình, không kết luận thiết kế logic D thiếu nhật ký.
- Hướng sửa: nêu tạo nhật ký Nhập/Điều chỉnh/Xuất bán cùng thay đổi tồn; xuất bán gắn MaHD và lượng gộp theo MaSP trên toàn hóa đơn, đúng MaCN. Chỉ bổ sung đặc tả GĐ1.

### LC-08 — Giờ hoạt động và quy tắc tự đóng chưa nối thành điều kiện nhất quán (A/C)

**Mức: cần làm rõ chính sách nội bộ đã tự chọn, không phải yêu cầu mới từ đề.**

- Theo QD15, mô phỏng hoạt động trong ngày; tự đóng lượt vào trước giờ đóng. Nhưng điều kiện QT-C1 chưa nói xử lý khi quét ngoài giờ; QT-C4 cũng chưa giới hạn khung PT trong giờ chi nhánh hoạt động.
- Ví dụ chi nhánh đóng 22h, cửa vẫn được đánh dấu hoạt động, hội viên quét 23h có thẻ hợp lệ: QT-C1 hiện không có điều kiện từ chối. QT-C3 không thể ghi giờ ra 22h vì phải sau 23h; tài liệu chỉ đưa vào danh sách bất thường, chưa nói cách xử lý phiên này. Phiên còn mở sẽ chặn lần vào tiếp theo theo C-R02.
- Hướng sửa tối thiểu cần được thống nhất: nhận quét vào trong giờ hoạt động và chỉ cung cấp khung PT nằm trọn trong giờ mở, hoặc nêu rõ ngoài giờ là ngoại lệ cần đối soát và quy tắc đóng phiên hợp lệ. Không tự thêm ca qua đêm/24 giờ. Việc này tách biệt với sức chứa: vẫn chỉ báo cáo, không chặn do lấp đầy.

## Những điểm không coi là lỗi hoặc yêu cầu GĐ2 còn thiếu

- PT tự trừ theo giờ, hủy trước >=2 giờ; quyền vào cổng độc lập; chỉ có cổng tính lượt; gia hạn Gym/Yoga nối tiếp; PT khung giờ tròn: đã khớp chính sách người dùng, không yêu cầu quay về cách cũ.
- 12 tháng gần nhất thay vì năm dương lịch đã chốt để giải quyết chỗ đề mô tả khác nhau; điểm giữ nguyên. Không cần mở lại quyết định này.
- Toàn lược đồ 2NF do lưu Hang là kết luận minh bạch theo thiết kế đã chốt. Đề yêu cầu xác định/chuẩn hóa, không ghi bắt buộc mọi bảng phải BCNF. Không tự bỏ Hang hoặc coi toàn B sai chỉ vì ngoại lệ được nêu.
- Chưa có index/partition/procedure, bộ dữ liệu >=70.000 dòng, 8–10 kịch bản vật lý, ma trận truy vấn hoặc benchmark <50 ms không phải lỗi thiếu đầu ra GĐ1 theo required.md và trang 4 đề.
- 5 vấn đề bắt buộc đã có nền thiết kế: quyền vào/log; khóa lịch và ràng buộc không trùng; dữ liệu báo cáo; cấu trúc điểm/hạng; lịch sử công tác/hoa hồng. Hiệu quả thực tế của chúng chưa được chứng minh, đúng phạm vi GĐ2.
- Báo cáo gói bán chạy theo số lượng ở chi nhánh được đề liệt kê là gợi ý. Hiện có doanh thu gói 6 tháng và sản phẩm bán chạy; có thể bổ sung phạm vi này khi nhóm muốn, không đánh thành lỗi bắt buộc của GĐ1.

## Chuẩn bị bản nộp, không tự điền thay nhóm

- Thiếu tên/MSSV của một số thành viên; chưa thấy nội dung đánh giá cá nhân/nhóm thực tế trong hồ sơ, mới có nhắc yêu cầu trong phân công. Đề trang 4 yêu cầu thông tin và đánh giá trong báo cáo tổng kết; cần chuẩn bị trước bản nộp tương ứng.
- README nhắc Issues nhưng chưa có đường dẫn cụ thể đến công cụ/bảng theo dõi. Chưa kiểm tra tài khoản/board trực tuyến nên không kết luận nhóm chưa dùng công cụ; cần đưa liên kết/minh chứng vào báo cáo theo yêu cầu chung.
- Ngưỡng hạng 20/50 triệu vẫn là giả định cần xác nhận với giảng viên; bảo lưu ngoài phạm vi hiện tại. Những điểm này đã được ghi rõ, không tự biến thành tính năng mới.

## Phạm vi kiểm tra và thứ tự sửa

Đã đọc đề PDF gốc, required.md, quyết định/phân công, các README, quy trình, tần suất, ERD hiện hành, thiết kế logic A–D và hai tài liệu bổ sung B, hồ sơ tổng. Các bản vẽ A drawio/C PNG cũ được hồ sơ đánh dấu lịch sử, không lấy làm thiết kế hiện hành. Kiểm tra lại 20 bộ thuộc tính ERD/lược đồ, 33 ánh xạ FK và đủ 52 nhãn tần suất đều đạt. Kiểm tra đó không thay cho các thiếu sót nội dung ở trên.

Ưu tiên xử lý LC-02/03/08 (thống nhất nghiệp vụ/cardinality), sau đó LC-01/04/07 (đủ quy trình/ràng buộc), cuối cùng LC-05/06 (đồng bộ bảng và bỏ nội dung cũ). Giữ mã LC-01…LC-08 để theo dõi, liên kết với GD1-10/11/13/15/17; không xóa hoặc đổi số 17 mục gốc. Lần rà này chỉ tạo báo cáo và cập nhật trạng thái, chưa sửa các quy tắc nghiệp vụ.
