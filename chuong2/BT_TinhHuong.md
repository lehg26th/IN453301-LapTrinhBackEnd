# Nghiên cứu trang 34: Tình huống "Hai người giành chỗ cuối"

Tình huống "Hai người giành chỗ cuối" mô tả một lỗi kinh điển trong quản trị cơ sở dữ liệu gọi là **điều kiện tranh chấp (race condition)**. Qua tình huống này, ta thấy rõ tầm quan trọng của tính **Cô lập (Isolation)** trong các giao dịch (Transaction).

## 1. Vấn đề xảy ra khi không có tính cô lập
* **Đọc dữ liệu đồng thời:** Phiên A (An) và Phiên B (Bình) cùng thực hiện câu lệnh `SELECT` để kiểm tra số chỗ trống của khóa học gần như cùng một lúc.
* **Đánh giá sai trạng thái:** Cả hai hệ thống đều nhận được kết quả trả về là "còn 1 chỗ".
* **Xung đột ghi:** Dựa trên kết quả vừa đọc được, cả hai phiên đều tiếp tục chạy lệnh `INSERT` để đăng ký và tiến hành `COMMIT` thành công.
* **Hậu quả:** Khóa học bị đăng ký vượt mức cho phép (ví dụ: 41/40 người). Lỗi này xảy ra do hai giao dịch thao tác dữ liệu xen kẽ nhau mà không bị ràng buộc hay biết về sự tồn tại của nhau.

## 2. Vì sao cần cô lập và cách xử lý
* **Vai trò của tính Cô lập (Isolation):** Tính cô lập (chữ "I" trong ACID) đảm bảo các giao dịch chạy đồng thời không can thiệp, không xung đột và không làm ảnh hưởng đến tính toàn vẹn dữ liệu của nhau.
* **Cách xử lý kỹ thuật:** Để giải quyết tình trạng tranh chấp này, cần sử dụng cơ chế **khóa hàng (Row-level locking)** bằng câu lệnh `SELECT ... FOR UPDATE` đặt bên trong một khối Transaction (`BEGIN; ... COMMIT;`).

### Ví dụ mô phỏng cách giải quyết
```sql
BEGIN;

-- 1. Đọc kiểm tra và khóa hàng dữ liệu của khóa học lại
SELECT COUNT(id) 
FROM registrations 
WHERE course_id = 1 
FOR UPDATE;

-- 2. Nếu số lượng < 40 (còn chỗ) thì mới thực hiện INSERT
INSERT INTO registrations (member_id, course_id) 
VALUES (id_hoc_vien, 1);

COMMIT;