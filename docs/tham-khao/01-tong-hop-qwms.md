# Tổng hợp chức năng và luồng nghiệp vụ Q-WMS

Nguồn: đủ 110 bài hướng dẫn tại https://help.best-inc.com/2/docs?language=zh-cn. Sáu bài chỉ có video; bài 658 (danh mục vài trăm trường tính phí) đọc khoảng hai phần ba đầu. Tài liệu gốc không mô tả chi tiết điều kiện của quy tắc cất hàng, phân bổ và sinh nhiệm vụ lấy hàng; các phần này phải tự thiết kế. Tên tiếng Trung giữ nguyên để đối chiếu.

Q-WMS (千易WMS) là hệ thống quản lý kho đa chủ hàng cho kho thương mại điện tử, gồm bản web cho quản lý, ứng dụng PDA cho nhân viên kho và cổng Portal cho chủ hàng.

## Tổng quan

ERP của chủ hàng đẩy chứng từ xuống WMS; kho xử lý nhập và xuất, mỗi luồng sáu bước; kết quả phản hồi ngược lên ERP.

- Nhập kho: phiếu nhập → hàng đến → kiểm hàng → nhận hàng → đóng phiếu → cất hàng. Tồn tăng khi nhận, bán được sau khi cất.
- Xuất kho: đơn xuất → gom đơn → in phiếu → lấy hàng → đóng gói → cân và giao. Tồn được giữ khi phân bổ, chỉ trừ khi giao.

## Danh mục module

| Module (menu) | Chức năng chính | Kênh |
| --- | --- | --- |
| 入库 Nhập kho | 入库单 phiếu nhập, 到达 xác nhận hàng đến, 验货 kiểm hàng, 收货 nhận hàng, 完成入库单 đóng phiếu, 上架 cất hàng, 退件收货 nhận hàng trả, 退件包裹管理 quản lý kiện trả | PC + PDA |
| 出库 Xuất kho | 订单 đơn xuất, 简易制单 tạo nhiệm vụ lấy hàng nhanh, 订单池 nhóm đơn và tạo đợt (波次), 拣货打单 in phiếu lấy hàng và vận đơn, 包装 đóng gói kiểm hàng, 称重发运 cân và giao, 拣货异常 ngoại lệ lấy hàng, 包裹存放管理 lưu kiện chờ lấy | PC + PDA |
| 库存 Tồn kho | 库存查询 tra tồn, 移库 chuyển vị trí, 库存调整 điều chỉnh và khởi tạo tồn, 实物盘点 kiểm kê, 差异处理 xử lý chênh lệch, 转移批次属性 đổi thuộc tính lô, 序列号管理 quản lý serial, 效期预警 cảnh báo hạn dùng | PC + PDA |
| 加工 Gia công | 加工单 lệnh lắp ráp, 加工拣货 lấy linh kiện, 加工 xác nhận, 加工上架 cất thành phẩm | PC + PDA |
| 任务 Nhiệm vụ | 任务列表 danh sách nhiệm vụ, 指派 giao cho người làm | PC |
| 数据 Dữ liệu nền | 货主 chủ hàng, 库区 khu, 库位 vị trí, 商品 hàng hoá (SKU) | PC |
| 用户 Người dùng | 用户 tài khoản, 角色 vai trò, 权限组 nhóm quyền | PC |
| 配置 Cấu hình | 参数配置 tham số theo chủ hàng, 流程配置 cấu hình quy trình, 流程规则 quy tắc phân bổ, 业务设置 thiết lập nghiệp vụ | PC |
| 工单 Phiếu việc | 退件工单 phiếu nhận lại hàng trả chưa rõ chủ | PC + Portal |
| 报表 Báo cáo | 盘点报表 báo cáo kiểm kê, 装运报告 báo cáo bàn giao | PC |
| 打印工具 In ấn | In mã SKU, mã vị trí, nhãn LPN, phiếu nhập, vận đơn qua plugin PET | PC |
| 结算 Tính phí | Yếu tố tính phí nhập, xuất, lưu kho, dịch vụ gia tăng, vận chuyển; hoá đơn tháng | PC + Portal |

Menu PDA được nêu trong tài liệu: 1 LPN收货验货, 2 上架 (gồm 3 按单号上架 và 5 直接上架), 3 拣货 (集中拣货, 边拣边分, 加工拣货), 7 移库, 8 库存 (实物盘点), 9 异常处理 (按订单补拣), 13 收货验货, 15 退货, và 发运 (包裹存放管理).

## Luồng nhập kho

Tồn chỉ thành tồn bán được sau khi cất hàng xong.

1. **Tra phiếu (入库单).** ERP đẩy phiếu xuống; kho tìm theo số phiếu ERP. Một số tra mờ được, nhiều số phải khớp chính xác.
2. **Xác nhận hàng đến (到达).** Nhập ngày đến, loại sức chứa (容量类型), số pallet, số thùng, vị trí tạm. Thêm hoạt động phụ (附加活动) như phí dỡ hàng, dán nhãn để tính phí. In phiếu nhập A4 và nhãn SKU nếu cần.
3. **Kiểm hàng (验货).** Quét SKU, chọn trạng thái tốt (良品) hay lỗi (不良品), xác nhận số lượng. Nếu SKU bật lô, hạn dùng hoặc serial thì nhập tại bước này. Có thể thu kích thước và cân nặng.
4. **Nhận hàng (收货).** Xác nhận lần hai; tồn tăng tại vị trí nhận.
5. **Đóng phiếu (完成入库单).** Dùng khi không còn hàng đến nữa. Không đảo ngược được.
6. **Cất hàng (上架).** Quét phiếu, xác nhận số lượng và vị trí đích. Trạng thái phiếu thành 已完成, giai đoạn cất hàng thành 上架完成.

### Các biến thể

| Cách làm | Dùng khi | Điểm khác |
| --- | --- | --- |
| PC nhập nhanh | Kho nhỏ, không có PDA | Mọi bước làm trên web; cất hàng có thể nhập từ Excel (SKU, số lượng, vị trí) |
| PDA nhận theo phiếu (13 收货验货) | Hàng rời | Quét số phiếu, quét SKU, serial nếu có, chọn tốt hay lỗi, rồi 全部接收 |
| PDA nhận theo LPN (1 LPN收货验货) | Hàng theo pallet, quy trình chuẩn | Quét LPN rồi SKU, nhập ngày sản xuất, hạn dùng, số lô; in nhãn LPN dán lên pallet; cất hàng theo LPN |
| PDA cất theo phiếu (按单号上架) | Sau khi đã nhận | Quét số phiếu nhận, SKU, số lượng, vị trí đích |
| PDA cất trực tiếp (直接上架) | Kho nhà máy, không có phiếu trước | Chọn chủ hàng, quét SKU và vị trí, bấm F3; hệ thống tự sinh phiếu nhập |

### Quy tắc kiểm tra khi cất hàng

- Vị trí phải tồn tại và đúng loại (lấy hàng, cách ly, lưu trữ, đặc biệt).
- Số loại SKU trong vị trí không vượt Max SKU; vị trí có kiểm soát vào ra.
- Quét mã thùng (CS) thì tự quy đổi ra số cái (EA). SKU bắt serial thì số serial quét phải bằng số lượng.
- Tham số IB008 quyết định có cho cất hàng chưa thu kích thước và cân nặng hay không.
- Hệ thống gợi ý vị trí và hiện vị trí cất lần trước.

### Cất trực tiếp trên PC (直接上架)

Giống bản PDA nhưng làm theo bảng: quét SKU, điền vị trí đích cho từng dòng, bấm cất hàng. Có nút điền vị trí gợi ý theo quy tắc cất hàng, tách một SKU ra nhiều vị trí, và nhập Excel. Phiếu nhập tự sinh có loại 其他入库, nguồn là WMS tự tạo, giai đoạn đã hoàn thành. Tham số GE006 quyết định khâu nào bắt thu serial.

## Luồng xuất kho

1. **Tra đơn (订单).** ERP đẩy đơn xuống; danh sách mặc định hiện đơn chưa hoàn thành, lọc theo nhiều điều kiện và lưu được bộ lọc nhanh.
2. **Gom đơn.** Hai đường:
   - 简易制单: lọc đơn theo cấu trúc đơn, chọn cách lấy hàng, bấm 生成拣货任务.
   - 订单池: đơn được phân vào cây phân loại theo điều kiện; chọn nút cây rồi 分类创建波次 để tạo đợt. Đợt phân bổ tồn rồi sinh nhiệm vụ theo quy tắc.
3. **In.** In phiếu lấy hàng hoặc nhãn lấy hàng và vận đơn; in lại tại 拣货打单 → 快速打印.
4. **Lấy hàng.** Theo phiếu giấy hoặc PDA. PDA tự nhận nhiệm vụ theo độ ưu tiên; quản lý có thể giao tay tại 任务列表 → 指派.
5. **Đóng gói kiểm hàng (包装).** Quét mã nhiệm vụ, LPN hoặc số đơn, quét từng SKU, chọn vật liệu đóng gói, xác nhận; hệ thống in vận đơn.
6. **Cân và giao (称重发运).** Chọn nhóm hãng vận chuyển và cửa giao, tạo phiên, quét từng vận đơn. Kết thúc phiên thì in báo cáo bàn giao (装运报告).

### Phân loại đơn và cách lấy hàng

| Loại đơn | Định nghĩa | Lấy hàng đề xuất | Kiểm hàng đề xuất |
| --- | --- | --- | --- |
| Đơn một món bán chạy (单件爆款) | 1 đơn 1 sản phẩm, từ 10 đơn cùng cấu trúc | Gom chung (集货拣货) | Kiểm lô: quét một đơn mẫu, in vận đơn cả lô |
| Đơn nhiều món bán chạy (多件爆款) | 1 đơn từ 2 sản phẩm, từ 10 đơn cùng cấu trúc | Gom chung | Kiểm lô |
| Đơn một SKU một món | Dưới 10 đơn cùng cấu trúc | Gom chung | Quét LPN rồi từng SKU |
| Đơn một SKU nhiều món | 1 SKU, từ 2 cái | Gom chung hoặc vừa lấy vừa chia | Quét LPN hoặc mã thùng luân chuyển |
| Đơn nhiều SKU | Nhiều SKU | Vừa lấy vừa chia (边拣边分) hoặc theo đơn (按单拣货) | Quét mã thùng luân chuyển |

### Ba cách lấy hàng trên PDA

- **Gom chung (集货拣货):** quét LPN xe lấy hàng, quét vị trí, quét SKU đến khi đủ số, cuối cùng quét vị trí bàn đóng gói. Một vòng cho cả nhiệm vụ.
- **Vừa lấy vừa chia (边拣边分):** gắn các thùng luân chuyển vào ô xe theo thứ tự, mỗi món lấy xong bỏ vào ô hệ thống chỉ. Cần xe có ô ngăn; hợp với SKU ít và nhỏ.
- **Theo đơn (按单拣货):** mỗi đơn một thùng, mỗi đơn một vòng. Hợp với đơn rất nhiều SKU hoặc hàng lớn.

### Đơn giao trong ngày (当日达)

Sàn gửi mã lấy hàng (取件码) qua ERP xuống WMS. Kho lọc theo nhãn đơn 当日达 để ưu tiên tạo nhiệm vụ, hoặc lập một nút riêng trong 订单池 và đưa lên đầu. Sau khi đóng gói và giao, PDA ghi kiện vào kệ chờ (包裹存放管理); tài xế đến thì nhập mã lấy hàng và xác nhận đã lấy. Kiện có hai trạng thái: 已登记 và 已取件.

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

## Trả hàng, huỷ đơn và ngoại lệ lấy hàng

### Trả hàng (退件收货)

Hàng đã giao rồi quay về kho được xử lý như phiếu nhập loại trả hàng. Phiếu trả đến từ hai nguồn: chủ hàng tạo trên ERP rồi đẩy xuống, hoặc kho tự nhận và WMS tự sinh phiếu rồi phản hồi lên ERP.

| Tình huống | Dấu hiệu | Cách xử lý |
| --- | --- | --- |
| Trả nguyên vận đơn (原单退回) | Mã vận đơn trùng đơn xuất gốc | Quét vận đơn, quét SKU, chọn trạng thái, 确认收货. WMS tự tạo phiếu nhập và tăng tồn; kho cất hàng |
| Trả đổi vận đơn, biết chủ hàng | Quét báo không tìm thấy phiếu | 登记退件包裹: ghi nhận kiện, chưa nhận hàng |
| Trả đổi vận đơn, chưa rõ chủ hàng | Như trên | 提交认领工单: tạo phiếu nhận lại. Chủ hàng vào Portal bấm 认领, rồi tạo phiếu trả trên ERP đẩy xuống kho |

Thứ tự bắt buộc là nhận, kiểm, cất. Hàng tốt và hàng lỗi cất ở vị trí khác nhau. PDA cất hàng trả tại 15 退货 → 按库位退货上架, theo vị trí gợi ý hoặc gộp theo SKU.

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

### Huỷ đơn và trả hàng về vị trí (还货上架)

Đơn có giai đoạn 未完成 và trạng thái 取消 nghĩa là đã huỷ nhưng tồn vẫn bị giữ; dòng đơn chuyển màu vàng. ERP coi tồn đã nhả, nên nếu kho không trả hàng về thì đơn sau sẽ phân bổ thất bại.

1. Lọc đơn theo giai đoạn 未完成 và trạng thái 取消.
2. Vào 包装, nhập số đơn; hệ thống bật hộp xác nhận trả tồn về vị trí.
3. Tham số OB025 chọn vị trí trả về: vị trí lấy hàng gốc (tồn bán được), vị trí cách ly (tồn không bán được, chờ kiểm rồi chuyển), hoặc vị trí EXTRA (tồn bán được, chờ chuyển).
4. Đơn chưa đụng tới hàng thì đóng thẳng; giai đoạn đơn thành 关闭.

### Ngoại lệ lấy hàng (拣货异常)

- **Thiếu khi đóng gói:** tại 包装 bấm 上报异常, in nhãn ngoại lệ dán lên đơn.
- **Thiếu khi lấy hàng:** trên PDA bấm báo thiếu, lấy nốt phần còn lại, hệ thống nhắc in nhãn ngoại lệ.
- **Lấy bù:** PDA 9 异常处理 → lấy bù theo đơn hoặc theo nhóm. Hệ thống vẫn gợi ý vị trí gốc để xác nhận lại.
- **Lấy bù vẫn thiếu:** tồn của SKU tại vị trí đó chuyển sang vị trí chênh lệch (coi như một lần kiểm kê) và phải xử lý chênh lệch sau.
- **Cả kho hết hàng:** báo chủ hàng rút đơn rồi chỉnh tồn tay, hoặc giao một phần (部分发运); tài liệu khuyên chỉ dùng giao một phần cho đơn B2B và đơn ngoại tuyến.

## Nghiệp vụ trong kho

| Nghiệp vụ | Mục đích | Cách làm chính |
| --- | --- | --- |
| Chuyển vị trí (移库) | Dời hàng giữa các vị trí | Ba kiểu trên PC và PDA: theo LPN, theo SKU kèm số lượng, theo cả vị trí. PDA có kiểu hai chặng theo nhiệm vụ dùng khi chuyển kho. Chuyển theo LPN cần khu bật giữ LPN |
| Điều chỉnh tồn (库存调整) | Tăng giảm tồn hằng ngày | Tạo phiếu điều chỉnh: chỉnh tồn có sẵn, thêm tồn mới, hoặc nhập Excel. Chọn tăng, giảm hoặc đặt tổng; chỉnh lượng khả dụng hoặc lượng đóng băng. Lưu là nháp, 保存并提交 mới đổi tồn |
| Khởi tạo tồn (初始化) | Đưa tồn thực vào khi mới chạy hệ thống | Nhập Excel; kế thừa được tuổi tồn cũ |
| Kiểm kê (实物盘点) | Khớp tồn sổ với tồn thực | Xem bên dưới |
| Xử lý chênh lệch (差异处理) | Dọn tồn ở vị trí Variance | Chọn dòng rồi 调整; cùng SKU có một dương một âm thì 抵消 |
| Đổi thuộc tính lô (转移批次属性) | Sửa trạng thái tốt sang lỗi, số lô, ngày sản xuất, hạn dùng | Tra tồn rồi chọn thuộc tính cần đổi |
| Gia công (加工单) | Lắp ráp SKU con thành thành phẩm | ERP tạo SKU lắp ráp và lệnh, đẩy xuống WMS. WMS: 分配 → 发放 → in phiếu xuất linh kiện → lấy hàng → xác nhận gia công → cất thành phẩm. Thành phẩm chỉ tăng tồn sau khi cất |

### Kiểm kê

1. Tạo phiếu tại 库存 → 实物盘点: loại theo vị trí hoặc theo hàng; chế độ sáng (明盘, thấy tồn sổ) hoặc tối (暗盘, ẩn tồn sổ, được khuyên dùng); độ ưu tiên bốn mức.
2. Thêm phạm vi: chọn cả khu và loại vị trí, chỉ định từng vị trí, hoặc chọn theo tồn (nhiều chủ hàng, SKU, khu, loại vị trí).
3. Bấm 发放 để phát nhiệm vụ.
4. PDA 8 库存 → 实物盘点: quét vị trí, quét SKU, nhập số đếm, lặp đến hết.
5. Trên PC xem kết quả. Muốn đếm lại thì đóng vòng hiện tại, tạo vòng kế tiếp, rồi phát lại.
6. Ghi nhận chênh lệch (记录差异), hoàn tất kiểm kê, xuất báo cáo.

### Kiểm kê: các quy tắc chính xác

- Kiểm kê không khoá kho: lấy hàng, chuyển vị trí, cất hàng vẫn chạy song song.
- Tồn sổ dùng để so là tổng tồn của SKU tại vị trí, gồm cả phần đã phân bổ và đã lấy.
- Chênh lệch bằng số đếm trừ tồn sổ tại đúng lúc bấm nộp, không phải lúc quét vị trí.
- Nộp số đếm không đổi tồn. Phải xác nhận chênh lệch rồi chạy ghi nhận chênh lệch (Record Variance) mới điều chỉnh.
- Cùng vị trí và SKU nộp nhiều lần trong một vòng thì lần sau đè lần trước. Đếm lại tối đa 10 vòng, chỉ cho các dòng chưa xác nhận.
- Chế độ thứ ba ngoài sáng và tối là 埋点: hiện số tham chiếu gây nhiễu để kiểm tra người đếm.
- Phiếu kiểm kê đi qua đã tạo, đã phát, đang làm, xong vòng đầu, đóng vòng, chờ đếm lại, đã đếm lại, hoàn thành; có thêm treo và huỷ.
- GE007 bắt nhập đủ lô và hạn dùng khi kiểm kê và khi ghi nhận chênh lệch. OB032 liên quan tới kiểm kê do báo thiếu khi lấy hàng.

### Vị trí chênh lệch (Variance)

Tồn vào vị trí Variance là dòng một chiều: bị đóng băng, không khả dụng, và sinh phiếu điều chỉnh loại 差异库存. Tồn vào đây khi lấy bù vẫn thiếu, đóng ngoại lệ giao thiếu, bổ sung hàng bị thiếu, hoặc xác nhận chênh lệch kiểm kê. Thiếu khi kiểm kê tạo số âm. Điều chỉnh xong sinh phiếu loại 差异调整.

### Bổ sung hàng (补货)

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

### Phiếu việc gia tăng (增值工单)

Dùng cho việc không đổi tồn và vị trí: đổi nhãn, đóng gói lại, dán nhãn, chụp ảnh. Phiếu có tiêu đề, loại, chủ hàng, người xử lý, danh sách SKU (chỉ để ghi nhận), và hoạt động phụ để tính phí. Nhiệm vụ đi qua phát, nhận, xử lý, treo, hoàn thành, đóng. Trên PDA có nhận và xử lý, chuyển giao, ghi phản hồi và chụp ảnh theo từng SKU.

Bảng định mức (物料清单) nối SKU cha với SKU con, dùng cho lệnh gia công lắp ráp và tháo rời.

### Bảng điều khiển (控制台)

Thẻ việc cần làm, bấm vào thì nhảy tới danh sách tương ứng; chỉ hiện thẻ có số khác 0, làm mới mỗi 10 phút. Các thẻ gồm: đơn quá hạn, đơn bị huỷ từ bên ngoài, phản hồi ERP thất bại, chủ hàng hết số dư, đơn và nhiệm vụ bị đóng băng, nhiệm vụ tạm dừng, ngoại lệ lấy hàng, chia hàng và đóng gói, chênh lệch nhận hàng trong ngày, chênh lệch quá hạn xử lý, trừ vật liệu đóng gói thất bại. Kèm biểu đồ số đơn giao trong 7 ngày.

## Dữ liệu nền và phân quyền

| Đối tượng | Trường và quy tắc chính |
| --- | --- |
| Chủ hàng (货主) | Mã khách duy nhất, mô tả, quốc gia, ERP liên kết; danh sách kho được gán; thuộc tính lô mặc định; giờ chốt đơn và SLA; danh sách hoạt động phụ; trạng thái 有效 hoặc 无效; cặp Access ID và Secret Key để ERP kết nối |
| Khu (库区) | Mã, mô tả, cờ giữ LPN. Lưu rồi không xoá và không đổi tên, chỉ đặt 无效 |
| Vị trí (库位) | Mã (ví dụ A01-01-22), khu, loại vị trí, trạng thái, Max SKU, kích thước. Tạo từng cái, sinh hàng loạt, hoặc nhập Excel |
| Vị trí nghiệp vụ trong khu TEMP | Nhận hàng, trả hàng, cách ly, gom phân loại, phân loại, bàn đóng gói, gia công, tạm |
| Hàng hoá (商品) | Do ERP đẩy xuống. Mã SKU, mã vạch EA và CS, số cái mỗi thùng, kích thước và cân nặng, cờ bắt serial, lô, hạn dùng, nước sản xuất |

Kho nhỏ đặt mọi vị trí là loại lấy hàng. Kho lớn tách vị trí lưu trữ và vị trí lấy hàng (存拣分离), vị trí lưu trữ làm nguồn bổ sung.

### Thuộc tính SKU

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

### Thuộc tính vị trí

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

### Lô, hạn dùng và serial

- Bật thu lô và hạn dùng ở chủ hàng, rồi ở từng SKU. Dữ liệu nhập ở bước kiểm hàng; một SKU hai ngày sản xuất thì nhận hai lần.
- Cảnh báo hạn dùng theo số ngày cấu hình. Quy tắc phân bổ có quy tắc con như hết hạn trước xuất trước, hoặc chỉ phân bổ hàng còn tối thiểu bao nhiêu ngày.
- Serial bật theo SKU và các tham số GE006, GE017, OB030. Khi bật, PDA nhập kho, nhận hàng trả và đóng gói đều bắt quét serial; đóng gói kiểm serial có đang trong kho không. PC kiểm hàng nhập không kiểm serial.

### Phân quyền ba lớp

- **Người dùng (用户):** tài khoản đăng nhập, ngôn ngữ, kho mặc định. Nhân viên tạm do quản trị đặt mật khẩu; nhân viên chính thức đăng ký qua email và buộc đổi mật khẩu mỗi 3 tháng. Lock mở lại được; Disable không tự bật lại được.
- **Vai trò (角色):** quyết định thấy trang và nút nào, tách quyền web, quyền PDA và quyền tính phí, chi tiết đến từng nút.
- **Nhóm quyền (权限组):** nối người dùng với vai trò, kèm phạm vi kho và chủ hàng, và độ ưu tiên nhiệm vụ PDA theo kho, khu hoạt động, loại nhiệm vụ. Một người vào nhiều nhóm, quyền cộng gộp; đổi quyền phải đăng nhập lại.

Nhóm dựng sẵn: quản lý kho, nhập kho, lấy hàng, đóng gói, giao hàng. Người tạo không cấp được quyền cao hơn quyền của mình.

## Trạng thái đơn và tham số cấu hình

| Mã | Trạng thái | Nghĩa |
| --- | --- | --- |
| 00 | 创建订单 | Đơn mới tạo |
| 09 | 取消 | Đã huỷ |
| 30 | 部分分配 | Phân bổ tồn một phần |
| 40 | 分配完成 | Phân bổ xong |
| 50 | 部分拣货 | Lấy hàng một phần |
| 60 | 拣货完成 | Lấy hàng xong |
| 61 | 部分验货 | Kiểm hàng một phần |
| 62 | 验货完成 | Kiểm hàng xong |
| 63 | 称重完成 | Cân xong |
| 70 | 部分发运 | Giao một phần |
| 80 | 完全发运 | Giao hết |
| 86 | 完全发货确认 | Đã xác nhận giao hết |
| 97 | 订单关闭 | Đơn đóng |

Đơn còn có trường giai đoạn riêng (未完成, 关闭) tách khỏi trạng thái. Phiếu nhập có giai đoạn đơn (đến 已完成) và giai đoạn cất hàng (đến 上架完成) theo dõi riêng.

| Mã | Tác dụng |
| --- | --- |
| GE005 | Quản lý tồn vật liệu đóng gói và trừ khi dùng |
| GE006 | Khâu nào bắt nhập và kiểm serial |
| GE007 | Kiểm chặt thuộc tính lô khi kiểm kê và ghi nhận chênh lệch |
| GE010, GE011, GE012 | Bật quay video, độ dài đoạn cắt, địa chỉ máy chủ FTP |
| IB002 | Cho phép nhận vượt số trên phiếu trả |
| IB008 | Cho phép hay không cất hàng chưa thu kích thước và cân nặng |
| IB009 | Tự tạo phiếu trả khi kiện khớp đơn xuất gốc |
| IB010 | Cho phép nhận SKU ngoài phiếu trả |
| OB013 | Kiểu giao hàng; chọn cân rồi giao khi nối máy phân loại |
| OB025 | Vị trí trả tồn về khi huỷ đơn đã lấy hàng |
| OB029 | Cách in khi đơn có nhiều tệp đính kèm |
| OB030, GE017 | Thu serial ở các khâu (cùng GE006) |
| OB032 | Kiểm kê do thiếu hàng và đưa hàng thiếu vào vị trí ngoại lệ |
| ST003 | Bật tính nhu cầu bổ sung theo mức tồn và theo đơn thiếu |

Tiền tố IB là nhập, OB là xuất, GE là chung, ST là thiết lập cấp kho. Năm nhóm quy tắc cấu hình được: cất hàng, phân bổ (có quy tắc con), sinh nhiệm vụ lấy hàng, tạo phiếu bổ sung, nhiệm vụ bổ sung. Tài liệu gốc chỉ nhắc tên các quy tắc này, không mô tả từng điều kiện.

### Nhãn hệ thống và màu đơn

Đơn và phiếu nhập mang nhãn do ERP gửi xuống hoặc do thao tác trong WMS sinh ra: đặt trước, sàn huỷ, giao trong ngày, chỉ định lô, nhận nhanh, lỗi lấy mã vận đơn, ngoại lệ lấy hàng, phân bổ và chia hàng, giao một phần, cập nhật vận đơn, có serial, kiện rỗng, hàng trả. Mỗi chủ hàng tự định nghĩa thêm nhãn riêng để gắn tay. Màu dòng: đỏ là đơn bị treo, vàng là đơn huỷ, tím là đơn đã giao nhưng bị huỷ trước khi hãng lấy.

### Hai trạng thái vận chuyển

Trạng thái vận chuyển (物流状态): chờ lấy, đã lấy, đang vận chuyển, đã ký nhận, đã huỷ, bất thường. Trạng thái lấy hàng (揽收状态) chỉ trả lời hãng đã lấy kiện hay chưa. Đã huỷ và chưa lấy nghĩa là hàng còn trong kho; đã huỷ và đã lấy nghĩa là phải truy lại kiện từ hãng. Hai trường này tách biệt với trạng thái đơn trong WMS.

## Tích hợp, thiết bị và tính phí

| Đối tác | Chiều dữ liệu | Nội dung |
| --- | --- | --- |
| ERP | ERP → WMS | Hàng hoá, phiếu nhập, đơn xuất, lệnh gia công, phiếu trả, mã lấy hàng của đơn giao trong ngày |
| ERP | WMS → ERP | Phản hồi phiếu nhập, đơn xuất, phiếu điều chỉnh, kết quả gia công, phiếu trả do kho tự tạo. Phản hồi lỗi phải được theo dõi |
| Portal chủ hàng | Hai chiều | Trạng thái đơn, chi tiết phí và hoá đơn tháng, số dư và nạp tiền, nhận lại phiếu hàng trả, tải lên phiếu nhập và đơn xuất, tra tồn |
| Máy phân loại và WCS | WCS → WMS | HTTP POST JSON. Gọi giao hàng với mã kho, mã vận đơn, cân nặng, dài rộng cao, người dùng, tên thiết bị; trả về mã hãng, số đơn, cờ thành công. Gọi thứ hai sinh biên bản bàn giao. Lỗi ký trả 401 |
| Hãng vận chuyển | WMS → hãng | Lấy mã vận đơn và in vận đơn; địa chỉ gửi lấy theo thứ tự đơn, chủ hàng, kho |

| Thiết bị | Bắt buộc | Dùng cho |
| --- | --- | --- |
| Máy in nhiệt | Có | Vận đơn, nhãn SKU, nhãn LPN, nhãn ngoại lệ |
| Máy in A4 | Không | Phiếu nhập, phiếu lấy hàng, phiếu đóng gói |
| PDA Android | Không | Nhận, cất, kiểm kê, lấy hàng, bổ sung; cần wifi phủ toàn kho |
| Súng quét | Không | Kiểm hàng trên máy tính |
| Cân điện tử cổng nối tiếp | Không | Cân kiện khi giao |
| Camera và máy chủ FTP | Không | Quay khâu đóng gói làm bằng chứng |
| Xe lấy hàng, thùng luân chuyển | Không | Gom chung hoặc vừa lấy vừa chia |

Định mức nêu trong tài liệu: một người kiểm khoảng 1.500 món mỗi giờ.

| Loại phí | Thời điểm ghi nhận | Yếu tố tiêu biểu |
| --- | --- | --- |
| Nhập kho | Lúc cất hàng xong | Số món nhận, số loại SKU, số pallet, số thùng, thể tích, cân nặng, loại phiếu, hoạt động phụ |
| Xuất kho | Ngày giao, hoặc ngày huỷ | Số món, số loại SKU, số kiện, số pallet, cân nặng và thể tích kiện, hãng vận chuyển, B2B hay B2C, vật liệu đóng gói, trạng thái trước khi huỷ, thu hộ |
| Lưu kho | Theo ảnh chụp tồn hằng ngày | Số món, thể tích, số LPN, loại vị trí, loại kệ, số ngày lưu, cỡ SKU chuẩn hay quá khổ |
| Dịch vụ gia tăng | Lúc tạo phiếu dịch vụ | Loại dịch vụ, số lượng, đơn vị |
| Vận chuyển | Theo vận đơn | Cước cơ bản |

### Thiết bị và camera

- PDA tự đăng xuất sau 3 giờ không thao tác; một tài khoản chỉ đăng nhập một PDA cùng lúc.
- Máy đóng gói dùng như một bàn đóng gói, không cần kết nối riêng. Máy giao hàng, cân đo thể tích và máy chia lần hai thì cần kết nối dữ liệu.
- Quay video: plugin PET tại bàn đóng gói quay khi nhận hàng trả trên PC và khi đóng gói, tải lên máy chủ FTP nội bộ, tra lại theo số đơn. GE010 bật tính năng, GE011 đặt số giây cắt trước và sau, GE012 đặt địa chỉ FTP.

### Hai thế hệ tính phí

Tài liệu mô tả hai thế hệ tính phí. Bản mới tách lớp rõ hơn và là bản nên tham khảo.

### Tính phí bản mới: chuỗi xử lý

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

## Gợi ý khi phát triển WMS tương tự

Phần này là đề xuất, không phải nội dung của tài liệu gốc.

### Thực thể dữ liệu cốt lõi

| Nhóm | Thực thể |
| --- | --- |
| Tổ chức | Kho, chủ hàng, gán chủ hàng với kho, người dùng, vai trò, nhóm quyền |
| Dữ liệu nền | Khu, vị trí, SKU, mã vạch và quy cách đóng gói (EA, CS), hãng vận chuyển, vật liệu đóng gói |
| Tồn kho | Dòng tồn theo khoá chủ hàng + SKU + vị trí + LPN + lô + trạng thái hàng, với ba lượng: tổng, đã phân bổ, đóng băng; serial; nhật ký giao dịch tồn |
| Nhập | Phiếu nhập, dòng phiếu, lần nhận, nhiệm vụ cất hàng, hoạt động phụ |
| Xuất | Đơn xuất, dòng đơn, đợt, phân bổ, nhiệm vụ lấy hàng, kiện, phiên giao, biên bản bàn giao |
| Trong kho | Phiếu chuyển, phiếu điều chỉnh, phiếu kiểm kê và các vòng đếm, lệnh gia công, ngoại lệ |
| Hệ thống | Tham số theo chủ hàng, cấu hình quy trình, quy tắc phân bổ, hàng đợi phản hồi ERP, yếu tố tính phí |

### Nguyên tắc thiết kế nên giữ

- Mọi biến động tồn đi qua một dịch vụ duy nhất và ghi nhật ký giao dịch; không màn nào sửa thẳng số tồn.
- Tách ba thời điểm của đơn xuất: phân bổ (giữ tồn), lấy hàng (dời tồn sang LPN hoặc bàn đóng gói), giao (trừ tồn).
- Huỷ đơn không tự nhả tồn đã lấy; phải có bước trả hàng về vị trí.
- Sai lệch không sửa tại chỗ mà dồn về vị trí chênh lệch, xử lý sau bằng phiếu điều chỉnh.
- Hành vi khác nhau giữa các chủ hàng nằm ở tham số và cấu hình quy trình, không viết cứng trong mã.
- PDA làm việc theo nhiệm vụ có độ ưu tiên; web dành cho lập kế hoạch, giám sát và xử lý ngoại lệ.
- Phản hồi lên ERP là bất đồng bộ, có thử lại và có màn theo dõi lỗi.

### Điều chỉnh đề xuất sau khi đọc đủ 110 bài

Các đề xuất này chưa áp vào `02-ke-hoach-xay-dung.md`.

- Mô hình vị trí cần đủ thuộc tính ngay từ giai đoạn 1: ưu tiên cất hàng, ưu tiên phân bổ, thứ tự lấy hàng, kiểm soát vào ra, Max SKU.
- Bổ sung hàng nên kéo từ giai đoạn 4 lên giai đoạn 3 nếu kho tách vị trí lưu trữ và vị trí lấy hàng.
- Hàng trả nặng hơn dự tính; nên tách thành hạng mục riêng trong giai đoạn 3 và làm luồng đơn giản trước.
- Màn đóng gói nên thiết kế theo bản mới; lấy và kiểm gộp một cho phép kho nhỏ chạy không cần PDA, hợp để chạy thử sớm ở giai đoạn 2.
- Tính phí theo mô hình bản mới; phí vận chuyển theo vùng có thể bỏ khỏi bản đầu. Ảnh chụp tồn hằng đêm nên có từ giai đoạn 1.
- Bảng điều khiển việc cần làm nên có từ giai đoạn 3.
