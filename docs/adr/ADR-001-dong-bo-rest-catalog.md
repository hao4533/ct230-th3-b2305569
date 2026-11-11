# ADR-001: Giao tiếp đồng bộ (REST) giữa order-service và catalog-service

## Trạng thái
Đã chấp nhận – 2026-11-11

## Bối cảnh
Đặt hàng cần giá sách từ catalog-service. Tuần trước, một lần catalog chậm 10 giây
làm toàn bộ luồng đặt hàng bị treo.

## Quyết định
Gọi đồng bộ bằng RestClient với timeout kết nối 1 s và timeout đọc 2 s, bọc trong
cầu dao Resilience4j "catalog". Lỗi hạ tầng trả 503 kèm Retry-After: 10; 404 của
catalog là lỗi nghiệp vụ, không tính vào cầu dao. POST /api/v1/orders dùng
Idempotency-Key để client gửi lại an toàn.

## Phương án đã xem xét
- Bất đồng bộ (hàng đợi/sự kiện): tách rời thời gian chạy, nhưng khách không có giá
  và kết quả ngay lúc đặt, độ phức tạp cao hơn (thứ tự, bù trừ, theo dõi).
- Sao chép dữ liệu giá sang order-service: không phụ thuộc lúc chạy, nhưng giá có
  thể cũ và phải đồng bộ dữ liệu giữa hai dịch vụ.

## Hệ quả
- Tích cực: giá chính xác tại thời điểm đặt, phản hồi ngay cho khách, lỗi không lan truyền.
- Tiêu cực: order-service phụ thuộc catalog-service lúc chạy; bù bằng timeout, cầu dao
  và idempotency. Khi cầu dao mở, người dùng nhận 503 thay vì đơn hàng.
