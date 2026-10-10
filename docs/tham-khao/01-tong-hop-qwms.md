# Tổng hợp chức năng và luồng nghiệp vụ Q-WMS

Nguồn: 39 bài hướng dẫn đầu tiên tại https://help.best-inc.com/2/docs?language=zh-cn. Phần bổ sung từ 71 bài còn lại nằm ở `02-bo-sung-71-bai.md`. Tên tiếng Trung giữ nguyên để đối chiếu với tài liệu gốc.

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

## Trả hàng, huỷ đơn và ngoại lệ lấy hàng

### Trả hàng (退件收货)

Hàng đã giao rồi quay về kho được xử lý như phiếu nhập loại trả hàng. Phiếu trả đến từ hai nguồn: chủ hàng tạo trên ERP rồi đẩy xuống, hoặc kho tự nhận và WMS tự sinh phiếu rồi phản hồi lên ERP.

| Tình huống | Dấu hiệu | Cách xử lý |
| --- | --- | --- |
| Trả nguyên vận đơn (原单退回) | Mã vận đơn trùng đơn xuất gốc | Quét vận đơn, quét SKU, chọn trạng thái, 确认收货. WMS tự tạo phiếu nhập và tăng tồn; kho cất hàng |
| Trả đổi vận đơn, biết chủ hàng | Quét báo không tìm thấy phiếu | 登记退件包裹: ghi nhận kiện, chưa nhận hàng |
| Trả đổi vận đơn, chưa rõ chủ hàng | Như trên | 提交认领工单: tạo phiếu nhận lại. Chủ hàng vào Portal bấm 认领, rồi tạo phiếu trả trên ERP đẩy xuống kho |

Thứ tự bắt buộc là nhận, kiểm, cất. Hàng tốt và hàng lỗi cất ở vị trí khác nhau. PDA cất hàng trả tại 15 退货 → 按库位退货上架, theo vị trí gợi ý hoặc gộp theo SKU.

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

### Vị trí chênh lệch (Variance)

Tồn vào vị trí Variance là dòng một chiều: bị đóng băng, không khả dụng, và sinh phiếu điều chỉnh loại 差异库存. Tồn vào đây khi lấy bù vẫn thiếu, đóng ngoại lệ giao thiếu, bổ sung hàng bị thiếu, hoặc xác nhận chênh lệch kiểm kê. Thiếu khi kiểm kê tạo số âm. Điều chỉnh xong sinh phiếu loại 差异调整.

## Dữ liệu nền và phân quyền

| Đối tượng | Trường và quy tắc chính |
| --- | --- |
| Chủ hàng (货主) | Mã khách duy nhất, mô tả, quốc gia, ERP liên kết; danh sách kho được gán; thuộc tính lô mặc định; giờ chốt đơn và SLA; danh sách hoạt động phụ; trạng thái 有效 hoặc 无效; cặp Access ID và Secret Key để ERP kết nối |
| Khu (库区) | Mã, mô tả, cờ giữ LPN. Lưu rồi không xoá và không đổi tên, chỉ đặt 无效 |
| Vị trí (库位) | Mã (ví dụ A01-01-22), khu, loại vị trí, trạng thái, Max SKU, kích thước. Tạo từng cái, sinh hàng loạt, hoặc nhập Excel |
| Vị trí nghiệp vụ trong khu TEMP | Nhận hàng, trả hàng, cách ly, gom phân loại, phân loại, bàn đóng gói, gia công, tạm |
| Hàng hoá (商品) | Do ERP đẩy xuống. Mã SKU, mã vạch EA và CS, số cái mỗi thùng, kích thước và cân nặng, cờ bắt serial, lô, hạn dùng, nước sản xuất |

Kho nhỏ đặt mọi vị trí là loại lấy hàng. Kho lớn tách vị trí lưu trữ và vị trí lấy hàng (存拣分离), vị trí lưu trữ làm nguồn bổ sung.

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

| Tham số | Tác dụng |
| --- | --- |
| IB008 | Cho phép hay không cất hàng chưa thu kích thước và cân nặng |
| OB013 | Kiểu giao hàng; chọn cân rồi giao khi nối máy phân loại |
| OB025 | Vị trí trả tồn về khi huỷ đơn đã lấy hàng |
| GE006, GE017, OB030 | Bắt thu serial ở các khâu |

Các tham số khác xem bảng trong `02-bo-sung-71-bai.md`.

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
