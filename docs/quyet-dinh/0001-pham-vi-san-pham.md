# 0001. Phạm vi sản phẩm

- Ngày: 2026-10-10
- Trạng thái: Đã chốt
- Người quyết định: chủ dự án

## Bối cảnh
Cần xác định đối tượng phục vụ trước khi thiết kế mô hình dữ liệu.

## Quyết định
- Kho dịch vụ đa chủ hàng (3PL): có Portal chủ hàng, tính phí, phân quyền theo chủ hàng.
- Hỗ trợ đơn B2C thương mại điện tử và B2B.
- Q-WMS là cơ sở tham khảo chức năng; khi vận hành sẽ điều chỉnh theo thực tế.

## Hệ quả
- Mọi bản ghi nghiệp vụ gắn kho và chủ hàng ngay từ đầu.
- Hành vi khác nhau giữa chủ hàng đặt ở tham số, không viết cứng.
- Chứng từ lưu sẵn số liệu cân đo và hoạt động phụ để tính phí về sau.
- Tham khảo luồng nghiệp vụ, không sao chép giao diện, tên gọi hay tài liệu của Q-WMS.
