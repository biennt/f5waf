# Tìm hiểu môi trường, ứng dụng, các thiết lập cần thiết ban đầu

Đầu tiên, cần đảm bảo rằng ứng dụng đang hoạt động tốt với F5 BIG-IP làm reverse-proxy hoặc load balancer. Nghĩa là, tối thiểu những mục sau đã được cấu hình và hoạt động tốt:

- Cấu hình DNS đã trỏ tới VIP trên F5 BIG-IP
- Một virtual server đã được cấu hình trên F5 BIG-IP (có thể là HTTP hoặc HTTPS tùy nhu cầu)
- Các cấu hình như pool, monitor, node đều đang hoạt động đúng
- Hệ thống đã provision (bật) module ASM,

Các bước trong tài liệu này sẽ chỉ tập trung vào phần cấu hình WAF.

Bước tiếp theo, cần cấu hình lưu log các vi phạm, trên giao diện quản trị web của F5 BIG-IP, vào menu `Security` > `Event Logs` > `Logging Profiles`, bấm vào nút `Create`

- Đặt tên cho profile, ví dụ `security_log_profile`
- Chọn `Application Security` (đối với tính năng WAF và API Protection)
- Chọn `DoS Protection` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `DoS Protection`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó
- Chọn `Bot Defense` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `Bot Defense`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó. Ngoài ra, đối với `Bot Defense`, cần chọn thêm ít nhất 1 loại request cần log, ví dụ `Untrusted Bot`, `Malicious Bot`, `Suspicious Browser`

Bấm vào `Create` để tạo log profile.
