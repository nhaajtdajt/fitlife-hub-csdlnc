# ĐỒ ÁN THỰC HÀNH — CSC12002 Cơ sở dữ liệu nâng cao

> Chuyển từ `2627-CSDLNC-DA.CLC.pdf` (4 trang) sang Markdown để agent đọc nhanh. Nội dung giữ nguyên theo đề; chỗ nào đề in đậm/đỏ được đánh dấu **QUAN TRỌNG**.

## 1. Thông tin chung

| Mục | Giá trị |
|---|---|
| Mã số đồ án | 2627.ADB.01 |
| Thời gian thực hiện | 10 tuần |
| Hình thức | Theo nhóm |
| Cách nộp bài | Qua hệ thống Moodle |
| GV phụ trách | Hồ Thị Hoàng Vy, Tiết Gia Hồng |
| Liên lạc | hthvy@fit.hcmus.edu.vn, tghong@fit.hcmus.edu.vn |

## 2. Chuẩn đầu ra môn học

- **G1** – Phân tích, tổng hợp, thực hành kỹ năng làm việc nhóm trong thiết kế CSDL quan hệ.
- **G3** – Thực hành suy nghĩ có phê phán, sáng tạo trong thiết kế CSDL quan hệ.
- **G5** – Vận dụng kiến thức nền tảng cho các giai đoạn thiết kế CSDL quan hệ: mô hình dữ liệu, PTH (phụ thuộc hàm), dạng chuẩn, ...
- **G6** – Vận dụng các kỹ thuật xác định yêu cầu người dùng.
- **G7** – Thực hiện thiết kế CSDL quan hệ ở mức quan niệm và logic.
- **G8** – Thực hiện thiết kế CSDL quan hệ ở mức vật lý.

## 3. Mô tả đồ án

### Mục tiêu

Sinh viên quan sát, hiểu cách vận hành một quy trình từ hệ thống thực tế, thu thập dữ liệu để tự thiết kế, cài đặt mô phỏng quy trình đó bằng kiến thức lý thuyết.

### Mô tả gợi ý: Hệ thống **FitLife Hub**

12 chi nhánh tại các thành phố lớn. Mỗi chi nhánh cung cấp: phòng tập (Gym / Yoga / GroupX), HLV cá nhân (PT), Spa/Sauna, cửa hàng bán thực phẩm bổ sung / phụ kiện tập luyện.

#### Chi nhánh
Mã CN, tên chi nhánh, địa chỉ, số điện thoại, giờ mở/đóng cửa, danh sách dịch vụ/tiện ích có tại chi nhánh.

#### Hội viên (Khách hàng)
Họ tên, SĐT, Email, CCCD, giới tính, ngày sinh, ngày đăng ký.
Một hội viên có thể mua **nhiều** gói tập (gói theo tháng/năm, gói PT 1-1, vé Spa lẻ).

#### Gói dịch vụ & Sản phẩm
- **Gói tập/Dịch vụ:** Mã gói, tên gói, loại (Gym, Yoga, PT 1-1, Spa), thời hạn (3 / 6 / 12 tháng), số buổi PT (nếu có), giá tiền.
- **Sản phẩm bán lẻ:** Mã SP, tên SP, loại (Whey protein, bình nước, thảm yoga), giá bán, **số lượng tồn kho theo từng chi nhánh**.

#### Lịch hẹn & Check-in
- Mỗi lần hội viên đến tập tại **bất kỳ** chi nhánh nào, hệ thống ghi nhận lượt **Check-in**: Mã check-in, Hội viên, Chi nhánh, Thời gian vào, Thời gian ra, Cửa kiểm soát/Thiết bị.
- Với gói **PT 1-1**, hội viên phải đặt **Lịch hẹn PT** trước: HLV phụ trách, Ngày giờ tập, Chi nhánh, Trạng thái (`Đã đặt` / `Đã hoàn thành` / `Hủy`).

#### Hóa đơn & Thanh toán
Mã hóa đơn, ngày lập, nhân viên lập, hội viên, danh sách gói dịch vụ/sản phẩm, tổng tiền, giảm giá, hình thức thanh toán (`Tiền mặt`, `VNPay`, `Thẻ`).

#### Chương trình Hội viên thân thiết
- 3 hạng: **Silver – Gold – Platinum**, dựa trên **tổng chi tiêu trong năm**.
- Tích điểm Loyalty: **1 điểm = 100.000 VNĐ**.

#### Quản lý Nhân sự
- Chức vụ: HLV (PT), Lễ tân, Quản lý chi nhánh, Nhân viên bán hàng.
- Thông tin: Mã NV, họ tên, SĐT, chức vụ, chi nhánh hiện tại, lương cơ bản.
- Nhân viên có thể có **lịch sử điều chuyển công tác** giữa các chi nhánh.

### Vấn đề hệ thống thực tế (BẮT BUỘC giải quyết)

Hệ thống hiện tại gặp 5 vấn đề nghiêm trọng. Nhóm **bắt buộc** phải phân tích và đưa ra giải pháp thiết kế (ở cấp độ Schema, Index, Partition, Stored Proc, Trigger, ...).

| # | Vấn đề | Mô tả | Nhiệm vụ sinh viên |
|---|---|---|---|
| 1 | **Nghẽn Check-in giờ cao điểm** | Khung 17h00–20h00 hằng ngày có > 10.000 lượt quét thẻ/QR check-in đồng thời toàn hệ thống. Truy vấn xác thực hội viên (kiểm tra hạn gói tập + lưu log check-in) bị đơ, gây ùn tắc cửa quay. | Đề xuất cấu trúc bảng/Index hoặc kỹ thuật lưu trữ/partition log check-in sao cho tra cứu quyền vào cổng **< 50ms**. |
| 2 | **Xung đột đặt lịch HLV cá nhân** | Hai hội viên cùng lúc đặt cùng một PT vào cùng khung giờ; do giao dịch đồng thời, dữ liệu bị ghi đè → PT bị trùng lịch dạy. | Đề xuất cơ chế ràng buộc **ngăn chặn tuyệt đối** Overbooking. |
| 3 | **Bùng nổ dữ liệu Log & Báo cáo doanh thu chậm** | Bảng Check-in và lịch sử tập luyện tăng > 300.000 dòng/tháng. Báo cáo *Tỷ lệ lấp đầy phòng tập theo khung giờ/chi nhánh* hoặc *Doanh thu theo tháng* mất > 1 phút. | Đề xuất **phân vùng dữ liệu** và thiết kế **bảng tổng hợp** hoặc View/Trigger để tối ưu báo cáo. |
| 4 | **Tính lại hạng hội viên bị quá tải** | Mỗi lần thanh toán hóa đơn mới, hệ thống lại cộng tổng tiền hóa đơn 12 tháng qua để nâng/hạ hạng (Silver/Gold/Platinum) và tính điểm thưởng → làm chậm thao tác in hóa đơn. | Thiết kế lại **cấu trúc lưu trữ điểm/hạng** để thanh toán không bị nghẽn. |
| 5 | **Lịch sử điều chuyển nhân sự & Lương/Hoa hồng sai lệch** | PT có thể chuyển từ Chi nhánh A sang B trong tháng. Khi tính hoa hồng PT (dựa trên số buổi dạy tại từng chi nhánh), truy vấn xác định *tại thời điểm buổi tập đó PT thuộc chi nhánh nào* bị sai. | Thiết kế lược đồ quản lý **Lịch sử công tác/Điều chuyển** để truy vấn hoa hồng chính xác và nhanh. |

## 4. Yêu cầu truy vấn / thống kê gợi ý

### Cấp Chi nhánh
- Doanh thu theo ngày / tháng / quý / năm.
- Danh sách hội viên check-in trong ngày theo khung giờ.
- Thống kê gói tập / sản phẩm bán chạy nhất tại chi nhánh.
- Báo cáo tồn kho thực phẩm bổ sung / phụ kiện.
- Tra cứu lịch sử tập luyện / lịch hẹn PT của một hội viên.
- Hiệu suất làm việc của HLV (số buổi dạy, điểm đánh giá từ hội viên).
- Thống kê hội viên sắp hết hạn gói tập hoặc lâu ngày chưa quay lại.

### Cấp Công ty / Ban giám đốc
- Doanh thu toàn hệ thống và so sánh giữa các chi nhánh.
- Gói dịch vụ mang lại doanh thu cao nhất trong 6 tháng gần nhất.
- Tỷ lệ phân bổ hạng hội viên (Silver / Gold / Platinum).
- Quản lý nhân sự toàn hệ thống (điều chuyển, cập nhật lương, tính hoa hồng).

## 5. Yêu cầu nộp bài

### Giai đoạn 1 — Phân tích, mức quan niệm & logic
- Tự mô tả chi tiết quy trình, dữ liệu, ràng buộc liên quan.
- Danh sách chức năng cần xây dựng **kèm tần suất giao dịch** tương ứng (sinh viên tự quan sát, phân tích, đề xuất); có thể bổ sung chức năng và ràng buộc chức năng.
- Thiết kế dữ liệu mức **quan niệm** và **logic**.

### Giai đoạn 2 — Mức vật lý & cài đặt
- Bảng thiết kế CSDL mức **vật lý** theo đề xuất của nhóm.
- Các đề xuất cải thiện hiệu quả truy vấn dựa trên thiết kế.
- Source code project mô phỏng một số chức năng (tìm kiếm, thêm/xóa/sửa, thống kê) có tần suất **cao và thấp**, bao gồm các kịch bản minh họa hiệu quả của giải pháp cải thiện hiệu suất.
- Source script: tạo CSDL, query, index, partition, ... (phải thực sự được sử dụng).
- Tạo, thực thi các chỉ mục; **chụp hình và giải thích kết quả**; **so sánh có / không** áp dụng giải pháp nâng cao hiệu suất.

### Lưu ý
- Báo cáo ghi rõ thông tin cá nhân, thông tin nhóm, đánh giá cá nhân và nhóm. Báo cáo là tài liệu tổng kết từ phân tích → thiết kế → cài đặt → đánh giá kết quả giải pháp.
- **QUAN TRỌNG:** Sinh viên tự phát sinh dữ liệu (kiểm tra dữ liệu khóa ngoại) đủ lớn để kiểm chứng các đề xuất chỉ mục — **≥ 70.000 dòng/bảng** cho một số bảng cần minh họa. Tự chọn công cụ phát sinh hoặc tự code.
- **QUAN TRỌNG:** Nhóm cần dùng **1 công cụ quản lý phân công và theo dõi tiến độ** nhóm.
- Tham khảo: https://www.guru99.com/test-data-generation-tools.html

### Thang điểm

| Phần | Tỉ trọng | Nội dung |
|---|---|---|
| Mức quan niệm | 20% | Đặc tả, thiết kế ER, báo cáo |
| Mức logic | 10% | Chuyển lược đồ quan hệ, xác định ràng buộc, dạng chuẩn, chuẩn hóa lược đồ, báo cáo |
| Mức vật lý | 50% | Script CSDL, stored procedure, function, trigger, phân tích tần suất, chỉ mục, partition, báo cáo, ... — **mô tả 8–10 kịch bản sử dụng có liên quan nhau**, xác định tần suất các truy vấn liên quan, áp dụng lý thuyết phân tích mức vật lý để chọn giải pháp phù hợp |
| Cài đặt ứng dụng | 20% | Web/WinForm, báo cáo |

**Tất cả thành viên** đều phải tham gia vào mọi giai đoạn (quan niệm, logic, vật lý, cài đặt).
