# Bổ sung từ 71 bài còn lại của tài liệu Q-WMS

Chỉ ghi phần mới so với `01-tong-hop-qwms.md`. Đã mở đủ 71 bài; sáu bài chỉ có video, bài 658 (danh mục vài trăm trường tính phí) đọc khoảng hai phần ba đầu. Tài liệu gốc không mô tả chi tiết điều kiện của quy tắc cất hàng, phân bổ và sinh nhiệm vụ lấy hàng; các phần này phải tự thiết kế.

## Dữ liệu nền

### Hàng hoá (SKU)

SKU do ERP đẩy xuống; SKU tạo riêng trong WMS (ví dụ vật tư tiêu hao) không đồng bộ ngược lên ERP. Mã SKU không sửa được sau khi tạo. Nhập Excel tối đa 3.000 dòng một lần, mẫu nhập tuỳ chỉnh được cột.

| Cờ trên SKU | Khi bật |
| --- | --- |
| Quản lý lô (CareBatch) | Nhập phải có số lô, xuất phải chỉ định lô, tồn tách theo lô |
| Quản lý hạn dùng (CareExpDate) | Nhập phải có hạn dùng, cảnh báo sắp hết hạn, hàng hết hạn không xuất được |
| Bắt serial (RequireSN) | Nhập và xuất đều quét serial, tra được chứng từ theo từng serial |
| Hàng bộ (Kit) | Xuất thì tách thành SKU con, tồn quản tới SKU con |
| Pin, nguy hiểm, chất lỏng, dễ vỡ | Dùng cho điều kiện lưu trữ, vận chuyển và tính phí |
| Cho phép không quét | Đóng gói không bắt quét SKU này |
| Là vật liệu đóng gói | Tự sinh một bản ghi vật liệu đóng gói loại thùng |

Cờ lô, hạn dùng, serial chỉ bật được trên SKU khi chủ hàng đã bật. Mã vạch trong cùng một chủ hàng không được trùng giữa các SKU và các đơn vị. SKU ngừng dùng thì không nhập xuất được nhưng tồn hiện có không đổi. Mỗi SKU có tab riêng cho quy cách (包装), mã vạch (条码), mức tồn bổ sung (配置) và nhật ký thay đổi.

### Vị trí

| Thuộc tính | Tác dụng |
| --- | --- |
| Max SKU | Số loại SKU tối đa; khuyên 1 cho vị trí lấy hàng, 3 đến 5 cho vị trí lưu trữ |
| Putaway Priority | Ưu tiên cất hàng, số nhỏ trước |
| Allocation Priority | Ưu tiên phân bổ, số nhỏ trước |
| Pick Sequence | Thứ tự đi lấy hàng; để trống thì so mã vị trí từng ký tự theo 0-9 rồi A-Z |
| Access Control | Trống: bình thường. In only: chỉ nhập. Out only: chỉ xuất. Blocked: khoá cả hai |
| LPN Req Flag | Kế thừa từ khu; bật thì cất hàng phải quét LPN |
| Forward Pick Loc | Đánh dấu vị trí từng được đặt làm vị trí lấy hàng cố định |

Vị trí đang có tồn không đổi được loại. Vị trí ngừng dùng vẫn xuất được tồn hiện có nhưng không cất thêm. Nhập Excel vị trí theo kiểu tất cả hoặc không: một dòng sai thì cả tệp thất bại. Cập nhật hàng loạt: ô trống giữ nguyên, ghi `delete` để xoá giá trị.

### Vật liệu đóng gói và hãng vận chuyển

- **Vật liệu đóng gói (包装材料):** mã, chủ hàng, loại (thùng, vật chèn, pallet), kích thước, cân nặng, tải trọng, giá, độ ưu tiên 1 đến 5, phạm vi chủ hàng được dùng (`ALLOWNER` là tất cả). SKU gán được vật liệu mặc định theo EA và CS để hệ thống gợi ý khi đóng gói. Bật GE005 thì vật liệu có tồn và bị trừ khi đóng gói.
- **Hãng vận chuyển (承运商):** ba kiểu kết nối là trực tiếp với hãng, qua nền tảng trung gian, và tự định nghĩa (không gọi API, dùng cho giao tự làm hoặc sàn chỉ định). Mỗi hãng có nhiều dịch vụ, loại giao là kiện (Parcel, cho B2C) hoặc hàng nặng (Freight, cho B2B). Hãng phải được gán vào kho mới dùng được. Nhóm hãng tối đa hai cấp, chỉ cho loại kiện, dùng ở bước cân và giao. Đổi hãng trên đơn thì phải lấy lại mã vận đơn.

## Nhập kho và hàng trả

### Cất trực tiếp trên PC (直接上架)

Giống bản PDA nhưng làm theo bảng: quét SKU, điền vị trí đích cho từng dòng, bấm cất hàng. Có nút điền vị trí gợi ý theo quy tắc cất hàng, tách một SKU ra nhiều vị trí, và nhập Excel. Phiếu nhập tự sinh có loại 其他入库, nguồn là WMS tự tạo, giai đoạn đã hoàn thành. Tham số GE006 quyết định khâu nào bắt thu serial.

### Hàng trả: hai chế độ làm việc

- **Đăng ký trước, nhận sau** (kho lớn): đăng ký kiện tại khu mở hàng, in nhãn, chuyển sang khu kiểm để nhận.
- **Quét vận đơn nhận luôn:** vào 退件收货, chọn hãng vận chuyển trước rồi quét; đăng ký không bắt buộc.

Đăng ký hàng loạt (极简批量登记) dùng khi hãng giao cả xe hàng trả: quét liên tục tối đa 500 mã một lần, hệ thống tự chia nhóm (khớp phiếu trả, khớp đơn gốc, không khớp, trùng, một mã khớp nhiều phiếu). Đăng ký chỉ ghi nhận hàng đã về kho, chưa tăng tồn.

### Bốn loại kiện trả

| Loại | Dấu hiệu | Bước tiếp |
| --- | --- | --- |
| Nhập được ngay (可入库件) | Khớp phiếu trả ERP đã đẩy xuống | Nhận hàng trả |
| Chờ tạo phiếu (待下单件) | Khớp đơn xuất gốc nhưng chủ hàng chưa tạo phiếu trả | Báo chủ hàng tạo phiếu, hoặc nhận luôn nếu bật IB009 |
| Chờ khách nhận (客户认领件) | Kho biết chủ hàng nhưng không khớp chứng từ nào | Chủ hàng xác nhận rồi tạo phiếu trả |
| Vô chủ (无主件) | Đổi vận đơn, không xác định được chủ hàng | Phát cho các chủ hàng trong kho nhận, ai nhận thì tạo phiếu |

### Hai tham số điều khiển hàng trả

| Tham số | Bật | Tắt |
| --- | --- | --- |
| IB009, theo chủ hàng | Kiện khớp đơn xuất gốc thì WMS tự tạo phiếu trả và phản hồi ERP; hợp với khách B2C nhiều đơn lẻ | Kiện không có phiếu phải đăng ký phân loại trước; hợp với khách thương hiệu kiểm soát chặt |
| IB010, theo kho | Quét được SKU ngoài phiếu, xoá được dòng đã quét (trả A nhập B) | Chỉ quét SKU có trong phiếu |

Nhận kiện rỗng (toàn bộ số kiểm bằng 0) không cần tham số: phiếu hoàn tất, gắn nhãn 空包, không tăng tồn, không sinh nhiệm vụ cất. Trả A nhập B: dòng gốc giữ số dự kiến và số nhận bằng 0, SKU mới thêm một dòng và tăng tồn đúng SKU thực nhận; nếu nhận vượt số thì cần thêm IB002.

### Trạng thái phiếu trả và rút lại

Phiếu đi qua Due in, Arrived, Inspecting, Receiving, Received, Closed. Rút lại được cho đến trước khi đóng phiếu, với điều kiện chưa phản hồi lên ERP và hàng chưa cất lại lên kệ.

### Ảnh bằng chứng

PDA chụp ảnh khi nhận hàng trả: chọn có hay không có bất thường, gắn SKU, ghi chú, nhiều ảnh; ảnh tự đóng dấu thời gian và số phiếu, lưu 15 ngày, xem lại trên PC và đồng bộ sang ERP.

## Xuất kho

### Nhóm đơn (订单池)

Cây phân loại hoạt động như một cái phễu: đơn đi từ nút trên xuống, khớp nút nào thì dừng ở đó, không khớp nút nào thì rơi vào nút "other". Điều kiện chặt nhất phải đặt trên cùng. Điều kiện ghép bằng "và" và "hoặc" theo khối. Nút đơn bán chạy đặt bằng cờ "chỉ gồm đơn cùng mặt hàng" kèm số đơn tối thiểu. Cây tự làm mới mỗi 5 phút.

Tạo đợt theo hai cách: mỗi nút một đợt (分类创建波次) hoặc gộp các nút đã chọn thành một đợt (合并创建波次). Chỉ đợt tạo từ nhóm đơn hoặc tạo tay mới chạy qua quy tắc sinh nhiệm vụ lấy hàng: quy tắc chính lọc đơn, quy tắc con lọc dòng đơn.

### Lấy hàng và chia lần hai

- **Xác nhận lấy hàng (拣货确认):** màn thực thi chung cho PC và PDA: nhận nhiệm vụ, quét vị trí nguồn, SKU, số lượng, LPN hoặc vị trí đích, báo thiếu, hoàn tất; có thao tác hoàn lại. Một nhiệm vụ không nên mở cùng lúc trên PC và PDA.
- **Sau khi báo thiếu:** chọn một trong hai lối tại màn ngoại lệ, lấy bù hoặc giao một phần, không trộn.
- **Chia lần hai (二次分拣):** dùng sau khi lấy gộp nhiều đơn. Quét bàn chia, quét LPN hoặc mã nhiệm vụ, gắn mỗi ô với một LPN của một đơn (phải gắn hết ô mới được quét hàng), rồi quét từng món để hệ thống chỉ ô, hoặc chọn ô trước rồi quét. Xong thì hàng chuyển sang bàn đóng gói. Hoàn lại được từng đơn.

### Kiểm hàng đóng gói bản mới (验货打包)

Một ô quét nhận nhiều loại mã, nhận diện theo thứ tự: mã nhiệm vụ, LPN, mã vận đơn, số đơn xuất, số đơn sàn. Nhiệm vụ nhiều đơn hiện thành tường ô đánh số: trắng là chưa bắt đầu, xanh nhạt đang kiểm, xanh lá đã đóng gói, đỏ nhạt có bất thường.

| Cấu hình | Cấp | Lựa chọn và hệ quả |
| --- | --- | --- |
| Chế độ kiểm hàng | Kho | Lấy và kiểm gộp một (拣验一体): quét đơn chưa lấy thì hệ thống tự dời tồn từ vị trí lấy hàng sang bàn đóng gói rồi cho kiểm ngay. Hoặc phải xác nhận lấy hàng trước. Gộp một không dùng cho nhiệm vụ chia lần hai |
| Vừa chia vừa kiểm (边分边验) | Kho | Lấy gộp nhiều đơn rồi chia ngay tại bàn đóng gói, bỏ qua bước chia lần hai |
| Kiểu chia | Cá nhân | Theo đơn: chọn ô rồi quét. Từng món: quét thẳng, hệ thống chọn đơn cần món đó nhất |
| Thu vật liệu đóng gói | Kho | Có hoặc không; gợi ý tự động (theo chỉ định của ERP, rồi gán trên SKU, rồi tự tính) hoặc chọn tay |
| Xác nhận vật liệu | Cá nhân | Tự xác nhận hoặc bấm xác nhận |
| Thu và kiểm serial | Kho | Bắt quét serial; tuỳ chọn kiểm serial đang trong kho. Đơn có SKU bắt serial không được đóng gói hàng loạt |
| Nguồn cân nặng | Kho | Cân nặng lý thuyết, nhập tay hoặc thiết bị |

Kiểm đủ số chưa phải đóng gói xong: phải bấm đóng gói (F2) mới sinh kiện, trừ tồn và kích hoạt in. Đóng gói hàng loạt (F4) chỉ cho đơn bán chạy một kiện. Quét thiếu thì kết thúc kiểm và ghi ngoại lệ thiếu hàng thực tế. Sau đóng gói, nếu bật tự giao thì hệ thống kiểm điều kiện giao ngay.

### Quét giao trên PDA (扫描发运)

Chọn loại kiện hoặc hàng nặng, quét mã vận đơn, mã pallet, số đơn hoặc số đơn sàn. Quét vận đơn thì giao từng kiện; quét số đơn thì giao mọi kiện của đơn. Tuỳ chọn xem trước rồi mới giao, và chụp ảnh sau khi giao. Không giao được khi kích thước kiện bằng 0, thiếu serial, hoặc đơn nhiều kiện đã giao dở mà quét số đơn. Có cấu hình giao nhanh cho phép giao từ lúc mới sinh nhiệm vụ lấy hàng.

### Đơn B2B

Đơn đến với loại B2B, kiểu giao hàng nặng, hãng loại Freight. Phân bổ theo thùng hoặc pallet. Lấy theo đơn, một đơn tách thành nhiều nhiệm vụ cho nhiều người. Kiểm hàng tại 出库验货 (khác màn đóng gói B2C), cấu hình được có đóng thùng và xếp pallet hay không. Giao bằng phiếu vận chuyển (运输单) trên PC hoặc giao theo pallet trên PDA.

### In ấn

- Đơn có cả vận đơn và hoá đơn: tham số OB029 chọn cho người dùng tự chọn tệp, chỉ tự in vận đơn, hoặc in cả hai cùng lúc.
- In vận đơn từ trình duyệt không cần plugin PET.
- Vận đơn in theo tệp gốc của hãng hoặc theo mẫu khi cần in thêm thông tin.

## Bổ sung hàng (补货)

Dời tồn từ vị trí lưu trữ sang vị trí lấy hàng, không sinh chứng từ ra ngoài. Có ba loại nhu cầu độc lập, cộng thêm tạo phiếu bổ sung bằng tay.

| Loại nhu cầu | Kích hoạt | Lượng bổ sung | Làm tròn theo thùng | Công tắc |
| --- | --- | --- | --- | --- |
| Theo mức tồn (水位补货) | Tồn khả dụng ở khu lấy hàng nhỏ hơn hoặc bằng mức tối thiểu; tác vụ định kỳ quét | Lên tới mức tối đa | Xuống, dưới 1 thùng thì bổ sung 1 thùng | ST003, mục mức tồn |
| Theo đơn vừa phân bổ (订单补货需求) | Phân bổ xong mà dòng đơn thiếu ở khu lấy hàng | Lượng thiếu ròng: tổng thiếu trừ tồn khả dụng khu lấy hàng trừ lượng đang bổ sung | Lên | Không cần |
| Theo tổng đơn chờ (缺货订单补货需求) | Gộp các đơn chờ xuất và phân bổ một phần theo chủ hàng và SKU; chạy định kỳ hoặc bấm tính lại | Tổng nhu cầu của các đơn, không phải phần thiếu hiển thị | Lên | ST003, mục đơn thiếu |

- Mức tồn đặt trên SKU (tính gộp cả khu lấy hàng) hoặc trên từng vị trí lấy hàng cố định (mỗi vị trí xét riêng). Có cả hai thì vị trí cố định được ưu tiên.
- Mức tối thiểu bằng 0 hoặc để trống thì SKU không vào danh sách.
- Khi tính nhu cầu, hệ thống chỉ nhìn tồn khả dụng. Khi tạo phiếu, tồn hiện có bằng tồn khả dụng cộng lượng đang bổ sung trên đường.
- Nguồn lấy từ khu lưu trữ theo quy tắc tạo phiếu bổ sung, xếp theo FIFO, LIFO hoặc hết hạn trước xuất trước; chỉ lấy hàng tốt.
- Làm mới tay giới hạn 10 phút một lần mỗi kho và không chạy cùng lúc với tác vụ định kỳ.

Phiếu bổ sung đi qua đã tạo, đã phát, đang làm, hoàn thành, hoặc đóng tay. Hai kiểu nhiệm vụ: bổ sung thẳng một lần, hoặc hai chặng qua vị trí trung chuyển. Bổ sung xong không tự giữ tồn cho đơn: phải chạy phân bổ lại. Đơn bị huỷ thì nhu cầu bổ sung của đơn đó tự xoá.

## Kiểm kê, phiếu việc và giám sát

### Kiểm kê: các quy tắc chính xác

- Kiểm kê không khoá kho: lấy hàng, chuyển vị trí, cất hàng vẫn chạy song song.
- Tồn sổ dùng để so là tổng tồn của SKU tại vị trí, gồm cả phần đã phân bổ và đã lấy.
- Chênh lệch bằng số đếm trừ tồn sổ tại đúng lúc bấm nộp, không phải lúc quét vị trí.
- Nộp số đếm không đổi tồn. Phải xác nhận chênh lệch rồi chạy ghi nhận chênh lệch (Record Variance) mới điều chỉnh.
- Cùng vị trí và SKU nộp nhiều lần trong một vòng thì lần sau đè lần trước. Đếm lại tối đa 10 vòng, chỉ cho các dòng chưa xác nhận.
- Chế độ thứ ba ngoài sáng và tối là 埋点: hiện số tham chiếu gây nhiễu để kiểm tra người đếm.
- Phiếu kiểm kê đi qua đã tạo, đã phát, đang làm, xong vòng đầu, đóng vòng, chờ đếm lại, đã đếm lại, hoàn thành; có thêm treo và huỷ.
- GE007 bắt nhập đủ lô và hạn dùng khi kiểm kê và khi ghi nhận chênh lệch. OB032 liên quan tới kiểm kê do báo thiếu khi lấy hàng.

### Phiếu việc gia tăng (增值工单)

Dùng cho việc không đổi tồn và vị trí: đổi nhãn, đóng gói lại, dán nhãn, chụp ảnh. Phiếu có tiêu đề, loại, chủ hàng, người xử lý, danh sách SKU (chỉ để ghi nhận), và hoạt động phụ để tính phí. Nhiệm vụ đi qua phát, nhận, xử lý, treo, hoàn thành, đóng. Trên PDA có nhận và xử lý, chuyển giao, ghi phản hồi và chụp ảnh theo từng SKU.

Bảng định mức (物料清单) nối SKU cha với SKU con, dùng cho lệnh gia công lắp ráp và tháo rời.

### Bảng điều khiển (控制台)

Thẻ việc cần làm, bấm vào thì nhảy tới danh sách tương ứng; chỉ hiện thẻ có số khác 0, làm mới mỗi 10 phút. Các thẻ gồm: đơn quá hạn, đơn bị huỷ từ bên ngoài, phản hồi ERP thất bại, chủ hàng hết số dư, đơn và nhiệm vụ bị đóng băng, nhiệm vụ tạm dừng, ngoại lệ lấy hàng, chia hàng và đóng gói, chênh lệch nhận hàng trong ngày, chênh lệch quá hạn xử lý, trừ vật liệu đóng gói thất bại. Kèm biểu đồ số đơn giao trong 7 ngày.

### Nhãn hệ thống và màu đơn

Đơn và phiếu nhập mang nhãn do ERP gửi xuống hoặc do thao tác trong WMS sinh ra: đặt trước, sàn huỷ, giao trong ngày, chỉ định lô, nhận nhanh, lỗi lấy mã vận đơn, ngoại lệ lấy hàng, phân bổ và chia hàng, giao một phần, cập nhật vận đơn, có serial, kiện rỗng, hàng trả. Mỗi chủ hàng tự định nghĩa thêm nhãn riêng để gắn tay. Màu dòng: đỏ là đơn bị treo, vàng là đơn huỷ, tím là đơn đã giao nhưng bị huỷ trước khi hãng lấy.

### Hai trạng thái vận chuyển

Trạng thái vận chuyển (物流状态): chờ lấy, đã lấy, đang vận chuyển, đã ký nhận, đã huỷ, bất thường. Trạng thái lấy hàng (揽收状态) chỉ trả lời hãng đã lấy kiện hay chưa. Đã huỷ và chưa lấy nghĩa là hàng còn trong kho; đã huỷ và đã lấy nghĩa là phải truy lại kiện từ hãng. Hai trường này tách biệt với trạng thái đơn trong WMS.

### Thiết bị và camera

- PDA tự đăng xuất sau 3 giờ không thao tác; một tài khoản chỉ đăng nhập một PDA cùng lúc.
- Máy đóng gói dùng như một bàn đóng gói, không cần kết nối riêng. Máy giao hàng, cân đo thể tích và máy chia lần hai thì cần kết nối dữ liệu.
- Quay video: plugin PET tại bàn đóng gói quay khi nhận hàng trả trên PC và khi đóng gói, tải lên máy chủ FTP nội bộ, tra lại theo số đơn. GE010 bật tính năng, GE011 đặt số giây cắt trước và sau, GE012 đặt địa chỉ FTP.

## Tính phí

Tài liệu mô tả hai thế hệ tính phí. Bản mới tách lớp rõ hơn và là bản nên tham khảo.

### Chuỗi xử lý của bản mới

1. **Thiết lập phí:** ba loại là phí thao tác, phí lưu kho, phí vận chuyển. Mỗi thiết lập gồm thông tin cơ bản (tiền tệ, đơn vị, độ chính xác, cách làm tròn), điều kiện áp dụng và giá.
2. **Phương án giá (报价方案):** gom các thiết lập phí, gắn với kho, khách, chiều thu hoặc trả, và khoảng ngày hiệu lực. Phải duyệt mới có hiệu lực.
3. **Phí nghiệp vụ (业务费用):** chứng từ phát sinh thì hệ thống tự khớp phương án còn hiệu lực và sinh bản ghi phí.
4. **Hoá đơn phí (费用账单):** gom phí nghiệp vụ theo khách, kho, chiều và khoảng ngày; xuất tệp với các cột tự chọn.

### Ba cách định giá

| Cách | Quy tắc | Ví dụ trong tài liệu |
| --- | --- | --- |
| Đơn giá cố định | Đơn giá nhân số lượng | 0,5 mỗi món, 200 món thành 100 |
| Theo khoảng (区间计费) | Tổng lượng rơi vào khoảng nào thì cả lượng tính theo giá khoảng đó | 3.500 lb rơi vào khoảng 1.000 đến 5.000 giá 0,25, thành 875 |
| Bậc thang luỹ tiến (区间阶梯累进) | Chia lượng ra từng khoảng rồi cộng dồn | 4.500 lb: 1.000 × 0,4 + 2.000 × 0,35 + 1.500 × 0,3 = 1.550 |

Khoảng mặc định bao gồm đầu trái và không gồm đầu phải; có số lượng tối thiểu tính phí. Trong bậc thang, mỗi khoảng có thể là đơn giá, tổng giá một lần, hoặc giá nối tiếp.

### Từng loại phí

- **Phí thao tác:** bốn nhóm là nhập, xuất, phiếu việc, gia tăng. Một phiếu có thể khớp nhiều hạng mục. Phí nhập ghi nhận khi cất hàng xong, phí xuất khi giao xong.
- **Phí lưu kho:** mỗi đêm hệ thống chụp ảnh tồn rồi tính phí ngày đó. Điều kiện thường dùng là tuổi tồn (ví dụ miễn 30 ngày đầu), loại vị trí, kho. Giá theo khoảng có thể khác nhau từng tháng.
- **Phí vận chuyển:** tính theo kiện: cước cơ bản theo vùng và khoảng cân nặng, cộng phụ phí nhiên liệu, mùa cao điểm, vượt cỡ, vùng xa. Phần này dành cho thị trường Mỹ; khách Đông Nam Á thường không dùng.

Điều kiện và cơ sở tính phí lấy từ một danh mục trường nghiệp vụ theo ngữ cảnh. Người dùng tạo thêm yếu tố riêng bằng công thức từ các trường, hằng số, bốn phép tính, Max và Min.

### Vận hành

- Sửa thiết lập phí chỉ ảnh hưởng chứng từ mới. Muốn áp lại cho chứng từ cũ thì chạy tính lại phí; tính lại ghi đè kết quả cũ.
- Bản ghi phí có thể được đánh dấu không tính phí, khôi phục, hoặc thêm hạng mục nhập tay.
- Bản ghi đã vào hoá đơn thì không được vào hoá đơn khác. Hoá đơn thu và hoá đơn trả tạo riêng.
- Tính lại, tạo hoá đơn và xuất tệp chạy nền, theo dõi trong lịch sử tác vụ.
- Phương án giá có phiên bản; đã duyệt thì không xoá, chỉ cập nhật thành phiên bản mới hoặc huỷ bỏ.

### Tài khoản trả trước

Số khả dụng bằng hạn mức tín dụng cộng số dư; bằng 0 hoặc âm thì khách không tạo được đơn mới. Khi đơn đẩy xuống WMS hệ thống tạm trừ, hoàn tất thì quyết toán theo phí thực. Khách nạp tiền trực tuyến hoặc chuyển khoản rồi gửi chứng từ để kho duyệt.

## Tham số mới gặp

| Mã | Tác dụng |
| --- | --- |
| IB002 | Cho phép nhận vượt số trên phiếu trả |
| IB009 | Tự tạo phiếu trả khi kiện khớp đơn xuất gốc |
| IB010 | Cho phép nhận SKU ngoài phiếu trả |
| GE005 | Quản lý tồn vật liệu đóng gói và trừ khi dùng |
| GE006 | Khâu nào bắt nhập và kiểm serial |
| GE007 | Kiểm chặt thuộc tính lô khi kiểm kê và ghi nhận chênh lệch |
| GE010, GE011, GE012 | Bật quay video, độ dài đoạn cắt, địa chỉ máy chủ FTP |
| OB029 | Cách in khi đơn có nhiều tệp đính kèm |
| OB032 | Kiểm kê do thiếu hàng và đưa hàng thiếu vào vị trí ngoại lệ |
| ST003 | Bật tính nhu cầu bổ sung theo mức tồn và theo đơn thiếu |

Năm nhóm quy tắc cấu hình được: cất hàng, phân bổ (có quy tắc con), sinh nhiệm vụ lấy hàng, tạo phiếu bổ sung, nhiệm vụ bổ sung.

## Tác động tới kế hoạch xây dựng

Đề xuất, chưa áp vào `03-ke-hoach-xay-dung.md`.

- Mô hình vị trí cần đủ thuộc tính ngay từ giai đoạn 1: ưu tiên cất hàng, ưu tiên phân bổ, thứ tự lấy hàng, kiểm soát vào ra, Max SKU.
- Bổ sung hàng nên kéo từ giai đoạn 4 lên giai đoạn 3 nếu kho tách vị trí lưu trữ và vị trí lấy hàng.
- Hàng trả nặng hơn dự tính; nên tách thành hạng mục riêng trong giai đoạn 3 và làm luồng đơn giản trước.
- Màn đóng gói nên thiết kế theo bản mới; lấy và kiểm gộp một cho phép kho nhỏ chạy không cần PDA, hợp để chạy thử sớm ở giai đoạn 2.
- Tính phí theo mô hình bản mới; phí vận chuyển theo vùng có thể bỏ khỏi bản đầu. Ảnh chụp tồn hằng đêm nên có từ giai đoạn 1.
- Bảng điều khiển việc cần làm nên có từ giai đoạn 3.
