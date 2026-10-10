# WMS Fulfillment

Hệ thống quản lý kho (WMS) cho kho dịch vụ đa chủ hàng (3PL), phục vụ đơn B2C thương mại điện tử và B2B.
Chức năng tham khảo từ Q-WMS, điều chỉnh theo thực tế vận hành của kho mình.

## Cấu trúc kho mã

| Thư mục | Nội dung | Nhóm phụ trách |
| --- | --- | --- |
| `docs/dac-ta/` | Đặc tả nghiệp vụ theo luồng, sơ đồ trạng thái, tiêu chí nghiệm thu | A |
| `docs/quyet-dinh/` | Nhật ký quyết định kiến trúc và nghiệp vụ | B |
| `docs/tham-khao/` | Tổng hợp Q-WMS, kế hoạch, phân công | A |
| `backend/` | Dịch vụ và API | C |
| `web/` | Web quản lý và Portal chủ hàng | D |
| `pda/` | Ứng dụng Android cho kho | E |
| `tich-hop/` | Kết nối nguồn đơn, hãng vận chuyển, in ấn | F |
| `ha-tang/` | Cấu hình triển khai, giám sát, sao lưu | H |

Kiểm thử (nhóm G) nằm cạnh mã của từng thư mục.

## Quy tắc làm việc

1. Đặc tả trong `docs/dac-ta/` là nguồn duy nhất. Đổi nghiệp vụ thì sửa đặc tả trước, rồi mới sửa mã.
2. Mỗi việc là một issue theo mẫu giao việc, đủ nhỏ để duyệt trong một ngày.
3. Làm trên nhánh riêng, mở pull request; không đẩy thẳng vào `main`.
4. Mọi thay đổi mã đi kèm kiểm thử tự động.
5. Người rà soát khác người (hoặc agent) viết; người phụ trách nhóm duyệt cuối.
6. Không lưu khoá, mật khẩu trong kho mã.
7. Mọi biến động tồn kho chỉ đi qua dịch vụ tồn kho trong `backend/`.
8. Quyết định quan trọng ghi vào `docs/quyet-dinh/`.

## Trạng thái

Giai đoạn 0 (chuẩn bị). Công nghệ và người phụ trách từng nhóm chưa chốt; xem `docs/quyet-dinh/`.
