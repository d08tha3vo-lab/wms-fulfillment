# Phân công agent và đội ngũ

Công việc chia thành tám nhóm; mỗi nhóm có một người phụ trách và một nhóm agent. Agent soạn đặc tả, viết mã và kiểm thử; người giao việc, quyết định, thử tại kho và duyệt trước khi gộp.

## Vòng làm việc

1. **Giao việc** (người phụ trách)
2. **Lập kế hoạch** (agent điều phối)
3. **Thực hiện** (agent chuyên môn)
4. **Rà soát** (agent độc lập)
5. **Duyệt và gộp** (người phụ trách); chưa đạt thì quay lại bước 3

Người phụ trách chỉ chạm vào đầu và cuối vòng, nên một người theo được nhiều việc chạy song song.

## Tám nhóm việc

| Nhóm | Agent làm | Người phụ trách làm | Sản phẩm |
| --- | --- | --- | --- |
| A. Nghiệp vụ và đặc tả | Soạn đặc tả từ ghi chép khảo sát, vẽ sơ đồ trạng thái, viết tiêu chí nghiệm thu và kịch bản thử, chỉ ra chỗ mâu thuẫn hoặc thiếu | Người phân tích nghiệp vụ hoặc quản lý kho: khảo sát tại kho, trả lời câu hỏi, chốt quy tắc | Bộ đặc tả theo luồng, lưu trong kho mã |
| B. Kiến trúc và dữ liệu | Đề xuất mô hình dữ liệu, hợp đồng API, so sánh phương án, ghi quyết định kiến trúc | Trưởng kỹ thuật: chọn phương án, duyệt mô hình tồn kho | Lược đồ dữ liệu, hợp đồng API, nhật ký quyết định |
| C. Backend lõi | Viết mã và kiểm thử theo từng module: dữ liệu nền, tồn kho, nhập, xuất, nhiệm vụ, kiểm kê | Lập trình viên backend: chia việc, duyệt mã, tự làm phần tồn kho đồng thời | Dịch vụ và API có kiểm thử tự động |
| D. Web và Portal | Dựng màn hình theo bản vẽ, nối API, viết kiểm thử giao diện | Lập trình viên web: duyệt mã, thử thao tác với người dùng kho | Web quản lý, sau đó Portal chủ hàng |
| E. Ứng dụng PDA | Viết màn hình quét, luồng nhiệm vụ, xử lý gửi lại khi mất mạng | Lập trình viên Android: thử trên PDA thật, chỉnh trải nghiệm quét | Ứng dụng Android cho kho |
| F. Tích hợp và in ấn | Viết bộ kết nối theo tài liệu API của đối tác, mẫu in, bộ giả lập đối tác để thử | Người giữ tài khoản đối tác: xin khoá, thử với hệ thống thật, thử máy in và cân | Kết nối nguồn đơn, hãng vận chuyển, in vận đơn |
| G. Kiểm thử và chất lượng | Viết kiểm thử tự động đầu cuối, rà soát mã độc lập, thử tải, soát bảo mật | Người kiểm thử: thử tại kho với hàng thật, quyết định đạt hay chưa | Bộ kiểm thử, báo cáo từng cổng |
| H. Hạ tầng và vận hành | Viết cấu hình triển khai, giám sát, kịch bản sao lưu và khôi phục | Người vận hành: giữ quyền môi trường chính thức và bí mật, bấm triển khai | Ba môi trường, quy trình triển khai, sổ tay sự cố |

Phần chỉ con người làm được: có mặt tại kho, cầm thiết bị, giữ khoá và mật khẩu, ký với đối tác, và chịu trách nhiệm về quyết định. Một người có thể phụ trách nhiều nhóm khi đội nhỏ; nhóm C và G không nên chung một người duyệt.

## Nhóm hoạt động theo giai đoạn

| Giai đoạn | Nhóm dẫn dắt | Nhóm tham gia | Việc chính của agent |
| --- | --- | --- | --- |
| 0. Chuẩn bị | A, B | H | Soạn đặc tả, mô hình dữ liệu, hợp đồng API, kịch bản thử; dựng khung kho mã và môi trường |
| 1. Nền và nhập kho | C | A, D, G, H | Module dữ liệu nền, phân quyền, tồn kho, nhập kho; màn web tương ứng |
| 2. Xuất kho B2C | C, F | A, D, G | Phân bổ, nhiệm vụ lấy hàng, đóng gói, giao; kết nối hãng vận chuyển đầu tiên; thử tải |
| 3. PDA và ngoại lệ | E | A, C, G, H | Ứng dụng PDA; huỷ đơn, thiếu hàng, hàng trả; chuẩn bị chạy thật |
| 4. Kiểm soát tồn và B2B | C, E | A, D, G | Kiểm kê, lô, serial, LPN, đợt, B2B, bổ sung hàng |
| 5. Portal và tính phí | D, F | A, C, G, H | Portal, yếu tố tính phí, hoá đơn, thêm kết nối |

Nhóm A chạy trước mỗi giai đoạn khoảng hai tuần để đặc tả sẵn sàng khi nhóm xây dựng bắt đầu. Nhóm G chạy suốt và giữ cổng nghiệm thu.

## Quy tắc phối hợp

- Đặc tả trong kho mã là nguồn duy nhất; đổi nghiệp vụ thì sửa đặc tả trước.
- Mỗi việc giao cho agent đủ nhỏ để một người duyệt xong trong một ngày, và có tiêu chí nghiệm thu viết sẵn.
- Mọi thay đổi mã đi kèm kiểm thử tự động; không có kiểm thử thì không gộp.
- Agent rà soát khác với agent viết mã; người phụ trách là người duyệt cuối.
- Agent không giữ khoá, mật khẩu và không tự triển khai lên môi trường chính thức.
- Mọi biến động tồn kho chỉ đi qua dịch vụ tồn của nhóm C.
- Quyết định quan trọng và lý do được ghi vào nhật ký quyết định để agent ở phiên sau đọc lại.
- Họp ngắn hằng tuần: xem việc đã gộp, việc bị chặn, câu hỏi agent đang chờ người trả lời.

## Việc cần để bắt đầu

- [ ] Gán tên người phụ trách cho từng nhóm A đến H
- [x] Tạo kho mã nguồn mà agent truy cập được, có thư mục đặc tả và nhật ký quyết định
- [ ] Chốt công nghệ cho backend, web, Android và cơ sở dữ liệu
- [ ] Gửi ghi chép khảo sát kho thí điểm để nhóm A soạn đặc tả luồng nhập kho trước
- [x] Thống nhất mẫu giao việc: mục tiêu, phạm vi, tiêu chí nghiệm thu, đặc tả liên quan
