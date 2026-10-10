# Kế hoạch xây dựng WMS 3PL đa chủ hàng

Một giai đoạn chuẩn bị và năm giai đoạn xây dựng, đưa kho thí điểm vào chạy thật sau giai đoạn 3, khoảng tuần 30; tổng cộng khoảng 46 tuần. Thời lượng là ước tính cho đội 7 người làm toàn thời gian và sẽ đổi theo quy mô đội thực tế.

## Lộ trình

| Giai đoạn | Ước tính | Tuần | Phạm vi | Điều kiện qua cổng |
| --- | --- | --- | --- | --- |
| 0. Chuẩn bị | 6 tuần | 1–6 | Khảo sát kho thí điểm, đặc tả nghiệp vụ, sơ đồ trạng thái, mô hình dữ liệu tồn kho, bản vẽ màn hình, chọn công nghệ, dựng môi trường | Đặc tả và mô hình dữ liệu được quản lý kho duyệt; kịch bản thử cho luồng nhập và xuất đã viết |
| 1. Nền và nhập kho | 8 tuần | 7–14 | Đa kho, đa chủ hàng, người dùng và phân quyền ba lớp, khu, vị trí, SKU; dịch vụ tồn kho có nhật ký; phiếu nhập, hàng đến, kiểm, nhận, cất; điều chỉnh và khởi tạo tồn; in mã vị trí và SKU | Nhập trọn một phiếu thật trên web; tồn theo vị trí khớp với đếm tay; hai chủ hàng không thấy dữ liệu của nhau |
| 2. Xuất kho B2C | 8 tuần | 15–22 | Nhận đơn qua API và Excel; phân bổ tồn; tạo nhiệm vụ lấy hàng; phiếu lấy hàng giấy; đóng gói kiểm hàng; lấy mã và in vận đơn của một hãng; cân và giao; phản hồi kết quả | Xử lý trọn 100 đơn thử từ nhận đơn đến bàn giao; tồn không âm khi nhiều người thao tác cùng lúc |
| 3. PDA và ngoại lệ | 8 tuần | 23–30 | Ứng dụng Android: nhận, cất, lấy hàng gom chung, chuyển vị trí; huỷ đơn và trả hàng về vị trí; thiếu hàng và lấy bù; vị trí chênh lệch; hàng trả; bảng kiểm hằng ngày | Kho thí điểm chạy thật một chủ hàng trong 2 tuần, sai lệch tồn dưới ngưỡng đã thống nhất |
| 4. Kiểm soát tồn và B2B | 8 tuần | 31–38 | Kiểm kê nhiều vòng; lô và hạn dùng; serial; LPN; nhóm đơn và đợt; vừa lấy vừa chia; kiểm lô đơn bán chạy; đơn B2B giao theo pallet và giao một phần; bổ sung hàng | Kiểm kê toàn kho bằng PDA; một đơn B2B và một đợt B2C hơn 200 đơn hoàn tất trong ngày |
| 5. Portal và tính phí | 8 tuần | 39–46 | Portal chủ hàng: trạng thái đơn, tồn, tải đơn, phiếu nhận lại hàng trả; yếu tố tính phí, bảng giá, hoá đơn tháng, nạp tiền trước; thêm hãng vận chuyển và kết nối ERP hoặc sàn | Xuất hoá đơn tháng đầu tiên cho một chủ hàng khớp với đối soát tay; chủ hàng thứ hai tự vận hành qua Portal |

Ba việc nên làm ngay từ giai đoạn 1 dù tính năng tương ứng đến sau: gắn chủ hàng vào mọi bản ghi, lưu hoạt động phụ và số liệu cân đo trên chứng từ để sau này tính phí, và đặt hành vi khác nhau giữa chủ hàng vào tham số.

## Việc chuẩn bị trước khi viết mã (giai đoạn 0)

**Nghiệp vụ**

- [ ] Chọn kho thí điểm và một chủ hàng đầu tiên, có người dùng chủ chốt phản hồi hằng tuần
- [ ] Đo khối lượng: đơn mỗi ngày, số SKU, số vị trí, số người thao tác cùng lúc, giờ cao điểm
- [ ] Viết luồng chuẩn cho nhập, xuất B2C, xuất B2B, trả hàng, huỷ đơn, kiểm kê
- [ ] Liệt kê ngoại lệ thường gặp và cách kho đang xử lý
- [ ] Thu mẫu chứng từ: phiếu nhập, phiếu lấy hàng, vận đơn, biên bản bàn giao, nhãn

**Đặc tả**

- [ ] Sơ đồ trạng thái của phiếu nhập, đơn xuất, nhiệm vụ, phiếu kiểm kê
- [ ] Quy tắc tồn kho: giữ, dời, trừ, đóng băng, chênh lệch
- [ ] Ma trận quyền theo vai trò cho web và PDA
- [ ] Danh sách tham số theo chủ hàng cho bản đầu
- [ ] Bản vẽ màn hình web và PDA cho luồng nhập và xuất
- [ ] Hợp đồng API nhận đơn và phản hồi

**Kỹ thuật**

- [ ] Mô hình dữ liệu tồn kho và nhật ký giao dịch
- [ ] Phương án xử lý đồng thời và chống ghi trùng khi PDA gửi lại
- [ ] Chọn công nghệ cho backend, web, Android, cơ sở dữ liệu, hàng đợi, in ấn
- [ ] Ba môi trường, triển khai tự động, sao lưu, giám sát
- [ ] Yêu cầu hiệu năng và thời gian ngừng cho phép

**Thiết bị và đối tác**

- [ ] Mua mẫu PDA, máy in nhiệt, súng quét, cân để thử
- [ ] Quy ước mã vị trí và in mã vạch cho kho thí điểm
- [ ] Mở tài khoản thử nghiệm với hãng vận chuyển đầu tiên
- [ ] Xác định nguồn đơn của chủ hàng đầu tiên: ERP, sàn hay Excel

## Đội ngũ giả định

| Vai trò | Số người | Việc chính |
| --- | --- | --- |
| Phân tích nghiệp vụ kiêm chủ sản phẩm | 1 | Đặc tả, ưu tiên, nghiệm thu với kho; cần hiểu vận hành kho |
| Backend | 2 | Dịch vụ tồn kho, chứng từ, API, tích hợp |
| Web | 1 | Màn quản lý, sau đó Portal |
| Android | 1 | Ứng dụng PDA, từ giai đoạn 3; trước đó hỗ trợ web |
| Kiểm thử | 1 | Kịch bản thử, thử tại kho |
| Hạ tầng và vận hành | 1 | Môi trường, triển khai, giám sát; có thể bán thời gian |

## Quyết định cần chốt trong giai đoạn 0

- Quy mô đội và ngân sách thực tế, để chỉnh lại thời lượng.
- Triển khai trên đám mây hay máy chủ đặt tại chỗ.
- PDA có cần làm việc khi mất mạng không; nếu có thì thiết kế ứng dụng khác hẳn.
- Nguồn đơn và hãng vận chuyển đầu tiên cần kết nối.
- Mức chi tiết tồn kho ở bản đầu: có theo lô và LPN ngay hay để giai đoạn 4. Mô hình dữ liệu nên có sẵn chỗ từ đầu.
- Cách tính phí với chủ hàng hiện nay, để biết cần lưu số liệu gì trên chứng từ.

## Rủi ro chính

- Sai lệch tồn do xử lý đồng thời kém: phòng bằng một dịch vụ tồn duy nhất và thử tải từ giai đoạn 2.
- Đặc tả xa thực tế kho: phòng bằng kho thí điểm và nghiệm thu tại chỗ ở mỗi cổng.
- Tích hợp hãng vận chuyển và nguồn đơn chậm hơn dự kiến: xin tài khoản thử từ giai đoạn 0.
- Chuyển đổi làm gián đoạn kho: kiểm kê trước, khởi tạo tồn, chạy song song vài ngày.
