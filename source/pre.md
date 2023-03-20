# Tìm hiểu môi trường, ứng dụng, các thiết lập cần thiết ban đầu

Trước hết, cần cấu hình lưu log các vi phạm, trên giao diện quản trị web của F5 BIG-IP, vào menu `Security` > `Event Logs` > `Logging Profiles`, bấm vào nút `Create`

Tại màn hình tiếp theo:
- Đặt tên cho profile, ví dụ `security_log_profile`
- Chọn `Application Security` (đối với tính năng WAF và API Protection)
- Chọn `DoS Protection` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `DoS Protection`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó
- Chọn `Bot Defense` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `Bot Defense`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó. Ngoài ra, đối với `Bot Defense`, cần chọn thêm ít nhất 1 loại request cần log, ví dụ `Untrusted Bot`, `Malicious Bot`, `Suspicious Browser`

Bấm vào `Create` để tạo log profile.
