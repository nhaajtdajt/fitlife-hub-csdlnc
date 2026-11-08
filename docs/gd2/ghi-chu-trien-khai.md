# Ghi chú triển khai GĐ2 tách từ rà soát GĐ1

Ngày 08/11/2026. Đây là hướng triển khai để xem xét, chưa cài đặt hoặc kiểm chứng bằng SQL Server. Các quy tắc nghiệp vụ, lược đồ và ràng buộc trong GĐ1 vẫn là căn cứ. Những trích đoạn cũ dưới đây được giữ để không mất ý; chúng không phải kết luận đã kiểm thử hay bắt buộc chọn đúng cơ chế đó.

## Xử lý PT theo đợt

Người dùng đồng ý hướng xử lý theo đợt. Gom các lịch đến giờ để xử lý cùng đợt; mỗi lịch hợp lệ vẫn trừ đúng một buổi ở gói liên kết, độc lập vào cổng và tham dự. Không cố ý gom lịch 17h để chờ 18h, không đợi cuối ngày. Lịch chưa đến giờ không được trừ sớm. Khi lỗi/chậm, xử lý bù mọi lịch đã đến giờ còn chưa trừ, không chỉ quét một cửa sổ giờ dễ bỏ sót. Chạy lại không trừ lần hai. Cỡ đợt, cách kích hoạt, chu kỳ kỹ thuật và độ trễ thực tế sẽ thiết kế/kiểm chứng ở GĐ2.

GĐ1 đếm đơn vị nghiệp vụ: A-F09 một buổi đối chiếu, CN-B08/C-F14 một lịch trừ buổi, C-F10 một lịch xác nhận vắng. Số lịch không phải số lần chạy tác vụ hoặc số câu truy vấn SQL. CN-B08 và C-F14 là cùng nghiệp vụ, không cộng hai lần.

## Nội dung kỹ thuật trước khi tách

Các đoạn có cả nghiệp vụ và kỹ thuật được lưu nguyên văn để truy vết; bản GĐ1 đã giữ lại yêu cầu nghiệp vụ tương ứng. Thứ tự khóa, SQL View, thủ tục và lịch chạy dưới đây cần được thống nhất lại khi triển khai và thử lỗi/đồng thời; không coi văn bản cũ là bằng chứng cơ chế đã đúng.

### Nguồn: docs/gd1/a/bang-tan-suat.md

> | A-F06 | Điều chuyển nhân viên | Cũ kết thúc hôm nay, mới bắt đầu ngày mai; ghi hai thay đổi cùng giao dịch | Đọc + ghi | ~10 lần/tháng | Giờ hành chính | Thấp | Đọc lịch PT chưa hủy bị ảnh hưởng, chặn nếu không phù hợp; cùng khóa nhân viên với đặt lịch. Ghi nguyên tử, chống chồng khoảng thời gian |

> GD1-10: chốt hoa hồng tạo một KyHoaHong/chi nhánh/tháng và một ChiTietHoaHong/buổi hoàn thành hợp lệ; xem/xuất kỳ đã chốt đọc dữ liệu này. Chốt chạy lại không ghi thêm; đo khóa kỳ khi ghi lịch và khóa cấu hình mức ở GĐ2.


### Nguồn: docs/gd1/a/quy-trinh.md

> 2. Trong giao dịch, khóa dòng NhanVien rồi đọc lại lịch sử và ngày hiện tại D theo Asia/Saigon. Ngày hiệu lực do hệ thống tính là D+1; hiển thị rõ ngày này khi thông báo kết quả.

> - Hai yêu cầu đồng thời: khóa cùng nhân viên; yêu cầu sau đọc lại và bị từ chối nếu yêu cầu trước đã tạo giai đoạn chờ.

> - Gọi E là 00:00 ngày D+1 theo Asia/Saigon. Sau khi khóa NhanVien, đọc lại các lịch của nhân viên có TrangThai khác Hủy và thời điểm kết thúc > E. Bao gồm lịch bắt đầu trước E nhưng còn kéo dài sau E; lịch kết thúc đúng E không bị ảnh hưởng.

> - Chống xung đột: đổi vị trí và đặt lịch cùng khóa dòng NhanVien trước khi đọc lại dữ liệu quyết định và giữ khóa đến khi kết thúc giao dịch. Đặt lịch khóa tiếp GoiHoiVien rồi LichHenPT; đổi vị trí chỉ đọc lịch, ghi lịch sử, không khóa/trừ gói. Không dùng kết quả kiểm tra trước khóa hoặc ảnh chụp dữ liệu cũ. Đặt lịch thành công trước thì đổi vị trí phải thấy lịch đó; đổi vị trí thành công trước thì đặt lịch phải thấy giai đoạn mới. Thủ tục/khóa SQL Server cụ thể cần kiểm thử ở GĐ2.

> - Tác vụ mỗi ngày lúc 00:00 Asia/Saigon khóa NhanVien, đọc lại ngày hiện tại và tìm đúng giai đoạn bao phủ ngày đó; cập nhật NhanVien.ChucVu và chi nhánh hiện tại nếu lưu trong cùng giao dịch. Chạy lại chỉ gán lại cùng giá trị, không tạo thêm lịch sử.

> - Kỳ đã chốt: không mở lại, sửa, xóa hoặc bổ sung buổi trong phạm vi hiện tại. Sửa mức hiện tại không đổi kỳ cũ. C dùng chung khóa kỳ khi cập nhật nguồn để không phát sinh buổi sau lúc chốt.


### Nguồn: docs/gd1/a/README.md

> - Thiết kế logic A đã hoàn thiện tại thiet-ke-logic.md (GD1-10). GD1-06 đã chốt: chưa cho đổi vị trí nếu còn lịch PT chưa hủy không phù hợp sau thay đổi; xem QT-A03. Đặt lịch và đổi vị trí cùng khóa nhân viên để tránh bỏ sót lịch mới.


### Nguồn: docs/gd1/a/thiet-ke-logic.md

> 1. Chỉ chốt tháng đã kết thúc. Xem trước là truy vấn, chưa ghi kỳ. Khóa theo (MaCN, Thang), kiểm tra chưa có kỳ; khóa/đọc mức chi nhánh nhất quán với thao tác sửa mức.

> 3. Tạo KyHoaHong với mức đã đọc và toàn bộ ChiTietHoaHong trong cùng giao dịch. Lỗi hoàn tác tất cả; tháng không có buổi hợp lệ vẫn được chốt với không dòng chi tiết. Chạy lại thấy khóa kỳ đã có thì trả kỳ cũ, không tính/lưu thêm.

> 6. Chốt kỳ và mọi thao tác C có thể thay đổi tập buổi/trạng thái trong kỳ dùng chung khóa (MaCN, Thang) trước các khóa nhân viên → gói → lịch hiện có; giữ tới commit, đọc lại sau khóa. Nếu di chuyển nguồn giữa kỳ thì phải khóa cả hai kỳ theo thứ tự cố định trước khi kiểm tra; luồng hiện tại không tự chuyển lịch. PK kỳ/PK mã buổi bảo vệ chốt lặp, nhưng không thay thế khóa chống buổi mới lọt vào lúc chốt. GĐ2 kiểm thử SQL Server thực tế.


### Nguồn: docs/gd1/b/bang-tan-suat.md

> | Đặt lịch PT: kiểm tra gói PT đủ hạn tại ngày hẹn và buổi chưa bị giữ (C) | Khóa nhân viên và xét lịch sử vị trí theo GD1-06; đọc/khóa gói theo MaHV/loại/hạn, đếm lại lịch Đã đặt/chưa trừ theo MaGHV trước khi thêm lịch | ~2.900 lần/ngày | **Cao** |


### Nguồn: docs/gd1/b/diem-hang.md

> Quy ước triển khai: mỗi khoản có mốc hết hạn bằng NgayLap cộng 12 tháng theo Asia/Saigon, giữ giờ; nếu thiếu ngày tương ứng thì kẹp cuối tháng. Khoản được tính khi NgayLap <= T < MocHetHan. Ví dụ 15/11/2026 10:00 hết hạn xét hạng tại 15/11/2027 10:00; ngày 29/02/2028 chuyển thành 28/02/2029 cùng giờ. Không xóa hóa đơn hoặc trừ điểm ở mốc này.

> 1. Thanh toán D và cập nhật B nằm trong cùng giao dịch. Khóa HoiVien theo MaHV, lấy thời điểm T và xử lý các dòng ghi nhận chưa loại nhưng có mốc hết hạn <= T: giảm TongChiTieu12T bằng TongTien của chính hóa đơn đó, bật DaLoaiKhoi12T cùng giao dịch. Không giảm DiemTichLuy.

> 4. Mọi bước lỗi hoàn tác cả hóa đơn, ghi nhận và tổng/điểm/hạng; không cho số dư âm. Cùng một hội viên luôn khóa HoiVien trước dòng ghi nhận, kể cả tác vụ nền và đọc hạng có đồng bộ. Khóa giữ đến hết giao dịch; PK MaHD bảo vệ ghi nhận một lần.

> Tác vụ mỗi đêm chọn khoản có DaLoaiKhoi12T = 0 đã hết hạn, xử lý theo từng hội viên bằng cùng giao dịch trên rồi cập nhật hạng. Chạy bù lấy tất cả khoản đến hạn còn chưa loại, không chỉ hóa đơn hết hạn hôm trước; chạy lại không trừ lần hai. Thanh toán đồng thời với tác vụ phải đọc lại sau khóa. Không quét/cộng toàn bộ hóa đơn 12 tháng trên đường thanh toán; chỉ xử lý phần đến hạn chưa được loại và hóa đơn mới. Chỉ mục phục vụ truy tìm khoản đến hạn/ghi nhận theo hội viên và giới hạn số học sẽ thiết kế, đo ở GĐ2.

> **Ánh xạ GD1-11:** GoiHoiVien cơ sở lưu MaGHV, MaHD, STT, DonVi, ngày và số dư; MaHV/MaGoi trong giao diện đọc trên đây lấy từ view vGoiHoiVien qua hóa đơn/dòng, không phải FK/cột lưu lặp trên gói. Lược đồ cơ sở và ERD hiện hành tại thiết kế B (đường dẫn tại nguồn: `thiet-ke-logic.md`) thay thế phần minh họa gói trước GD1-11; giữ nguyên quy tắc PT/điểm.


### Nguồn: docs/gd1/b/goi-pt.md

> Mọi thao tác đặt/hủy/trừ trên một gói giữ thứ tự khóa GoiHoiVien trước LichHenPT và đọc lại trạng thái trong cùng giao dịch. Riêng đặt lịch khóa NhanVien trước cả gói, đọc lại chức vụ/chi nhánh trong lịch sử gồm giai đoạn chờ để phối hợp đổi vị trí ở A (GD1-06). Hủy/trừ không lấy thêm khóa nhân viên sau khi khóa gói/lịch. Tra cứu ứng viên trước khi khóa chỉ là gợi ý; không được dùng số buổi đọc cũ để quyết định ghi. Không giữ kết quả đếm đã đọc trước khóa. Cách khóa cụ thể và kiểm thử SQL Server thuộc GĐ2.

> | Đặt | Thêm lịch có MaGHV, Đã đặt, DaTruBuoi = 0; không giảm số buổi | Khóa nhân viên, đọc lại điều kiện PT/chi nhánh; khóa gói rồi tính lại khả dụng, chủ sở hữu, loại/hạn gói và khung PT. Gói hết khả dụng thì thử ứng viên hợp lệ tiếp theo; xung đột khung PT thì rollback |

> | Hai yêu cầu cùng nhắm một buổi cuối | Sau khi khóa/đọc lại, chỉ một lịch giữ được buổi đó |

> **GD1-10 — kỳ hoa hồng:** thao tác có thể thay đổi tập lịch/kết quả theo chi nhánh–tháng phải lấy khóa kỳ trước các khóa nhân viên/gói/lịch hiện có; đọc lại KyHoaHong và từ chối thêm/đổi nguồn vào kỳ đã chốt. Không sửa/xóa MaNV, MaCN, NgayGio, TrangThai của buổi đã chốt. Chốt A và ghi nguồn C cùng dùng thứ tự này; xem thiết kế A (đường dẫn tại nguồn: `../a/thiet-ke-logic.md`).

> **Ánh xạ GD1-11:** GoiHoiVien cơ sở lưu MaGHV, MaHD, STT, DonVi, ngày và số dư; MaHV/MaGoi trong giao diện đọc trên đây lấy từ view vGoiHoiVien qua hóa đơn/dòng, không phải FK/cột lưu lặp trên gói. Lược đồ cơ sở và ERD hiện hành tại thiết kế B (đường dẫn tại nguồn: `thiet-ke-logic.md`) thay thế phần minh họa gói trước GD1-11; giữ nguyên quy tắc PT/điểm.


### Nguồn: docs/gd1/b/quy-trinh.md

> 3. B tính và D hiển thị ngày bắt đầu/hết hạn của từng gói trước khi hội viên xác nhận thanh toán. Khi ghi nhận, khóa phạm vi hội viên + loại gói và tính lại ngày nối tiếp để hai giao dịch mua đồng thời không tạo khoảng chồng nhau; nếu ngày dự kiến thay đổi, cần xác nhận lại trước khi hoàn tất thanh toán.

> 6. Đặt/hủy/trừ buổi giữ thứ tự khóa gói trước lịch; riêng đặt lịch khóa NhanVien trước cả gói để kiểm tra lại chức vụ/chi nhánh và phối hợp đổi vị trí (GD1-06); kiểm tra lại quyền lợi trong giao dịch để hai lần đặt đồng thời không dùng chung buổi cuối. Quy tắc dữ liệu và tình huống kiểm tra tại thiết kế gói PT (đường dẫn tại nguồn: `goi-pt.md`).

> | 5 | Liên kết lịch với đúng gói PT và giữ buổi khi đặt | LichHenPT.MaGHV; đếm lịch Đã đặt/chưa trừ trên từng gói, khóa khi ghi | Đã chốt và thiết kế GD1-03 |


### Nguồn: docs/gd1/b/thiet-ke-logic.md

> Không lưu lặp MaHV/MaGoi trên GoiHoiVien cơ sở: MaHD đã xác định MaHV, (MaHD,STT) đã xác định MaGoi; thêm hai cột đó sẽ có phụ thuộc vào một phần khóa ứng viên, không còn 2NF. View vGoiHoiVien nối GoiHoiVien → ChiTietHoaDon → HoaDon, công bố MaGHV, MaHV, MaGoi, NgayBatDau, NgayHetHan, SoBuoiConLai và khóa dòng/đơn vị. Đây là giao diện thay thế các danh sách cột GoiHoiVien có MaHV/MaGoi trong tài liệu PT/C trước GD1-11. Các lần khóa/trừ số dư vẫn dùng bảng cơ sở GoiHoiVien, không khóa một bản ghi gói khác qua view.

> HoiVien được giữ đúng phương án lưu tổng/điểm/hạng đã chốt. Phương án chuẩn hóa bỏ Hang khỏi bảng cơ sở và tính từ TongChiTieu12T qua view sẽ loại PTH không khóa; ở đây chủ động giữ Hang làm bộ nhớ tổng hợp, bắt buộc đồng bộ theo diem-hang.md. Không kết luận toàn B đạt BCNF. Không tạo bảng cho từng số tiền để chỉ che phụ thuộc TongChiTieu12T → Hang.

> 2. Trong một giao dịch, khóa HoiVien rồi bản ghi hóa đơn/dòng theo thứ tự ổn định, kiểm tra chủ hóa đơn, LoaiDong = GoiDichVu, MaGoi tồn tại/được bán và SoLuong nguyên dương. Một dòng sản phẩm tuyệt đối không tạo GoiHoiVien.

> 3. Dòng gói có SoLuong = n tạo đúng n bản ghi, DonVi lần lượt 1..n, mỗi bản ghi có MaGHV riêng. Liên kết (MaHD,STT) không đổi. Cùng dòng mua 2 PT tạo 2 gói riêng; cùng dòng mua 2 Gym TheoHan tạo 2 kỳ nối tiếp theo thứ tự STT/DonVi. Dùng khóa hội viên để tuần tự việc tính đuôi chuỗi, giữ quy tắc xác nhận lại ngày nếu dự kiến thay đổi.

> 6. Dòng và đầu hóa đơn đã thanh toán không sửa/xóa; gói không đổi liên kết/ngày/chủ/loại. Nếu có tác vụ B/D lấy đồng thời khóa hội viên và hóa đơn thì cùng thứ tự HoiVien → HoaDon → dòng; cập nhật quyền lợi C vẫn theo khóa kỳ/nhân viên/gói/lịch đã chốt. GĐ2 kiểm tra giao dịch, deadlock và lỗi giữa chừng thực tế.


### Nguồn: docs/gd1/c/bang-tan-suat.md

> **GD1-06:** mỗi lần đặt lịch phải khóa nhân viên và đọc lại lịch sử công tác, gồm giai đoạn chờ, trước khi khóa gói/ghi lịch. Đổi vị trí ở A đọc các lịch chưa hủy bị ảnh hưởng trong cùng cơ chế khóa. Đây là công việc bên trong giao dịch hiện có, không cộng thành lượt đặt lịch mới; đo chi phí tranh chấp khóa ở GĐ2.


### Nguồn: docs/gd1/c/business-rules.md

> - Số giữ chỗ = số lịch cùng MaGHV, TrangThai = Đã đặt, DaTruBuoi = 0, gồm cả lịch quá giờ chưa trừ. Đặt/hủy/trừ cùng khóa gói trước rồi khóa lịch và kiểm tra lại trong giao dịch.

> - Đặt lịch và đổi vị trí cùng khóa NhanVien trước rồi đọc lại lịch sử/lịch hẹn; đặt lịch sau đó khóa gói rồi lịch. Sau khi chấp nhận đổi vị trí, mọi lịch đặt mới phải phù hợp cả giai đoạn chờ.


### Nguồn: docs/gd1/c/erd.md

> - Số giữ chỗ, chọn gói đủ hạn, trừ đúng một lần và khóa giao dịch được mô tả trong thiết kế PT B–C (đường dẫn tại nguồn: `../b/goi-pt.md`).


### Nguồn: docs/gd1/c/quy-trinh.md

> 5. Nếu dùng vé một lượt, hệ thống khóa đúng vé và kiểm tra lại số lượt; kiểm tra lại phiên đang mở để chống quét lặp.

> 5. Trong giao dịch, khóa NhanVien của PT trước, đọc lại lịch sử công tác gồm cả giai đoạn chờ và kiểm tra toàn bộ buổi hẹn có chức vụ PT/đúng chi nhánh. Sau đó khóa gói đã chọn rồi đọc lại số buổi và lịch giữ chỗ, kiểm tra chủ sở hữu, loại gói, thời hạn, nơi công tác và khung giờ. Nếu gói vừa hết khả dụng thì chọn lại ứng viên hợp lệ; không đổi PT/khung. Tạo `LichHenPT` có `MaGHV`, trạng thái `Đã đặt`, `DaTruBuoi = 0`. Không lưu MaHV lặp trên lịch; hội viên suy ra qua gói. Thêm lịch làm tăng số giữ chỗ, chưa giảm số buổi. Mọi bước lỗi phải rollback. Giữ khóa nhân viên đến hết giao dịch để phối hợp với đổi vị trí ở A; không dùng kết quả kiểm tra PT trước khóa.

> 1. Hội viên chọn lịch `Đã đặt`; hệ thống lấy MaGHV bất biến của lịch, khóa gói trước rồi khóa và đọc lại lịch. Kiểm tra chủ sở hữu qua GoiHoiVien.MaHV, trạng thái và thời gian hiện tại.

> 2. Khóa GoiHoiVien theo MaGHV của lịch trước, rồi khóa và đọc lại lịch. Kiểm tra trạng thái/cờ, số buổi và hạn gói tại NgayGio của lịch (không phải ngày chạy bù). Không đọc CheckIn hoặc yêu cầu hội viên đã vào cổng.

> **GD1-10 — kỳ hoa hồng:** thao tác có thể thay đổi tập lịch/kết quả theo chi nhánh–tháng phải lấy khóa kỳ trước các khóa nhân viên/gói/lịch hiện có; đọc lại KyHoaHong và từ chối thêm/đổi nguồn vào kỳ đã chốt. Không sửa/xóa MaNV, MaCN, NgayGio, TrangThai của buổi đã chốt. Chốt A và ghi nguồn C cùng dùng thứ tự này; xem thiết kế A (đường dẫn tại nguồn: `../a/thiet-ke-logic.md`).

> **GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Khóa/trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem thiết kế B (đường dẫn tại nguồn: `../b/thiet-ke-logic.md`).


### Nguồn: docs/gd1/c/README.md

> - GD1-03 đã chốt MaGHV của từng lần mua. Lịch lưu MaGHV bắt buộc; đếm giữ chỗ từ lịch Đã đặt/chưa trừ trên từng gói, chọn hạn sớm nhất đủ ngày hẹn, khóa gói trước lịch khi ghi; riêng đặt lịch khóa nhân viên trước cả gói theo GD1-06. Chi tiết ở thiết kế PT B–C (đường dẫn tại nguồn: `../b/goi-pt.md`).

> **GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Khóa/trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem thiết kế B (đường dẫn tại nguồn: `../b/thiet-ke-logic.md`).


### Nguồn: docs/gd1/c/thiet-ke-logic.md

> | C-R07 | `LichHenPT`, `NhanVien`, `LichSuCongTac` | Nhân viên có ChucVu = PT trong đúng một khoảng LichSuCongTac bao phủ buổi tập tại MaCN của lịch; không dùng chức vụ hiện tại để xét lịch quá khứ/tương lai. Ngày công tác DATE tính cả hai biên theo #13. Đổi vị trí bị từ chối nếu làm lịch chưa hủy mất điều kiện này, kể cả buổi kéo qua mốc hiệu lực; đặt lịch và đổi vị trí cùng khóa NhanVien, đọc lại dữ liệu trong giao dịch. | `LichHenPT` (+ / − / + `MaNV, MaCN, NgayGio`); `NhanVien` (− / − / −); `LichSuCongTac` (+ / + / + `MaNV, MaCN, ChucVu, NgayBatDau, NgayKetThuc`) |

> - B/C dùng MaGHV đã công bố; kiểm tra/chọn gói, giữ chỗ, hủy và tự trừ tuân theo giao dịch PT (đường dẫn tại nguồn: `../b/goi-pt.md`). Mọi thao tác ghi giữ thứ tự khóa gói trước lịch, tính số giữ chỗ sau khóa. Riêng đặt lịch khóa thêm NhanVien trước gói để cùng tuần tự với thay đổi vị trí của A; hủy/trừ chỉ khóa gói rồi lịch, không lấy khóa nhân viên sau đó. Ràng buộc duy nhất khung PT vẫn phải kiểm tra đồng thời. Quyền vào cổng kiểm tra riêng; vé một lượt không liên quan giờ PT.

> **GD1-10 — kỳ hoa hồng:** thao tác có thể thay đổi tập lịch/kết quả theo chi nhánh–tháng phải lấy khóa kỳ trước các khóa nhân viên/gói/lịch hiện có; đọc lại KyHoaHong và từ chối thêm/đổi nguồn vào kỳ đã chốt. Không sửa/xóa MaNV, MaCN, NgayGio, TrangThai của buổi đã chốt. Chốt A và ghi nguồn C cùng dùng thứ tự này; xem thiết kế A (đường dẫn tại nguồn: `../a/thiet-ke-logic.md`).

> **GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Khóa/trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem thiết kế B (đường dẫn tại nguồn: `../b/thiet-ke-logic.md`).


### Nguồn: docs/gd1/d/quy-trinh.md

> 5. Khi thanh toán thành công, hệ thống trong một giao dịch: tạo `HoaDon`, tạo toàn bộ `ChiTietHoaDon`, trừ tồn kho theo số lượng, và gọi `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)` của B.


### Nguồn: docs/gd1/d/thiet-ke-logic.md

> GD1-12: ThanhTien là bí danh tính ra bằng SoLuong × DonGia trong SELECT/View khi đọc, không phải cột lưu hoặc cột computed persisted. Màn hình/in hóa đơn/phân bổ giảm giá dùng cùng biểu thức này. Giá danh mục hiện tại không tham gia; đơn giá chụp lúc bán và hóa đơn đã thanh toán vẫn bất biến. Không áp dụng kết luận BCNF của bảng cơ sở cho view đã thêm thuộc tính dẫn xuất.

> - B xác nhận thủ tục `sp_CapNhatDiemHang(@MaHV, @MaHD, @TongTien, @NgayLap)` và giao dịch tạo `GoiHoiVien` cùng quy trình thanh toán gói. Chữ ký đã có trong hợp đồng; chi tiết rollback liên module cần thống nhất trước triển khai.


### Nguồn: docs/quyet-dinh.md

> | 12 | PT có được dạy ở chi nhánh khác không? | Không, chỉ dạy tại chi nhánh đang công tác | A, C | Chốt GD1-06: PT chỉ dạy tại chi nhánh công tác và phải có chức vụ PT trong toàn bộ buổi hẹn. Đổi chức vụ/chi nhánh từ ngày mai bị từ chối nếu còn lịch chưa hủy bị thay đổi làm mất điều kiện này; trả danh sách lịch vướng, giữ nguyên dữ liệu. Không tự hủy, đổi HLV hoặc chuyển lịch. Đặt lịch và đổi vị trí cùng khóa nhân viên rồi đọc lại lịch sử/lịch hẹn trong giao dịch |


### Nguồn: docs/phan-cong.md

> | **A** | Khoảng công tác không chồng; nhân viên điều chuyển phải có chi nhánh đến khác chi nhánh đi; `NhanVien.ChucVu` đồng bộ từ giai đoạn bao phủ ngày hiện tại; bấm đổi ngày D thì lịch sử cũ kết thúc D, mới bắt đầu D+1, không đổi chức vụ hiện tại sớm; từ chối đổi nếu lịch chưa hủy mất điều kiện PT/chi nhánh, đặt lịch và đổi vị trí cùng khóa NhanVien | **Không phụ thuộc ai** |

> **GD1-11 — nguồn gói đã mua:** MaGHV vẫn là khóa bảng GoiHoiVien và đích FK của LichHenPT. MaHV/MaGoi trong giao diện B công bố được đọc từ vGoiHoiVien (nối dòng hóa đơn/đầu hóa đơn), không lưu lặp trên bảng gói cơ sở. Khóa/trừ số dư theo MaGHV vẫn tác động GoiHoiVien; giao diện đọc giữ các cột ngày và số dư đã công bố. Xem thiết kế B (đường dẫn tại nguồn: `gd1/b/thiet-ke-logic.md`).


### Nguồn bổ sung: docs/gd1/b/quy-trinh.md

> Khóa HoiVien trước HoaDon/dòng, ghi hóa đơn/gói/điểm/kho cùng commit hoặc rollback.

### Nguồn bổ sung: docs/gd1/d/quy-trinh.md

> Khóa HoiVien trước HoaDon/dòng, ghi hóa đơn/gói/điểm/kho cùng commit hoặc rollback.

### Nguồn bổ sung: docs/gd1/d/thiet-ke-logic.md

> Khóa HoiVien trước HoaDon/dòng, ghi hóa đơn/gói/điểm/kho cùng commit hoặc rollback.

## Đầu vào tải cần làm rõ trước kiểm thử — từ GD1-16

Đề ghi >10.000 lượt quét đồng thời toàn hệ thống trong 17h–20h. Giữ yêu cầu này; xác định cách thể hiện yêu cầu đồng thời trước khi thiết kế phép thử, không dùng trung bình 0,926 lượt/giây của mốc minh họa G1 thay thế. Phân biệt tải vào/ra, quét lỗi/lặp, phân bố chi nhánh, tốc độ đỉnh và số yêu cầu chồng thời gian. Mục tiêu <50 ms là tra cứu quyền vào, cần tách khỏi thời gian toàn luồng khi đo. Bộ số liệu GĐ1 là giả định tần suất, chưa phải kịch bản kiểm thử và chưa chứng minh hiệu năng.
