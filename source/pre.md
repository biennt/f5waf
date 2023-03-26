# Tìm hiểu môi trường, ứng dụng, các thiết lập cần thiết ban đầu

Đầu tiên, cần đảm bảo rằng ứng dụng đang hoạt động tốt với F5 BIG-IP làm reverse-proxy hoặc load balancer. Nghĩa là, tối thiểu những mục sau đã được cấu hình và hoạt động tốt:

- Cấu hình DNS đã trỏ tới VIP trên F5 BIG-IP
- Một virtual server đã được cấu hình trên F5 BIG-IP (có thể là HTTP hoặc HTTPS tùy nhu cầu)
- Các cấu hình như pool, monitor, node đều đang hoạt động đúng
- Hệ thống đã provision (bật) module ASM,

Ví dụ sau đây sử dụng giao diện dòng lệnh của F5 BIG-IP để tạo một cấu hình dịch vụ với các thông tin:

- Web server ở địa chỉ 10.1.10.11 port 80 (HTTP)
- F5 BIG-IP WAF được cấu hình với địa chỉ Virtual IP là 10.1.20.10 port 80
- F5 BIG-IP WAF sẽ monitor trạng thái của web server theo địa chỉ và port ở trên với phương thức `HTTP GET /` (nếu có mã 200 trả về là UP)
- F5 BIG-IP WAF sẽ tự lựa chọn địa chỉ Self IP của mình để kết nối tới Web server (Automap)

Các lệnh cần tạo trong TMSH shell như sau (từ Bash shell gõ tmsh để vào chế độ gõ các lệnh dưới đây, và sau đó gõ lệnh quit để quay lại Bash shell sau khi lưu cấu hình)

```
create ltm node 10.1.10.11
create ltm pool http_pool members add { 10.1.10.11:80 } monitor http
create ltm virtual vs_http destination 10.1.20.10:80 ip-protocol tcp pool http_pool profiles add { http } source-address-translation { type automap }

save sys config
```
Để kiểm tra, từ máy trạm sử dụng trình duyệt có thể kết nối tới địa chỉ http://10.1.20.10

Các bước trong tài liệu này sẽ chỉ tập trung vào phần cấu hình WAF.

Bước tiếp theo, cần cấu hình lưu log các vi phạm, trên giao diện quản trị web của F5 BIG-IP, vào menu `Security` > `Event Logs` > `Logging Profiles`, bấm vào nút `Create`

- Đặt tên cho profile, ví dụ `security_log_profile`
- Chọn `Application Security` (đối với tính năng WAF và API Protection)
- Chọn `DoS Protection` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `DoS Protection`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó
- Chọn `Bot Defense` (đối với tính năng Layer 7 Dos Protection)
Nếu chọn mục `Bot Defense`, cần chọn ô `Local Publisher` trong tab tương ứng hiện ra sau đó. Ngoài ra, đối với `Bot Defense`, cần chọn thêm ít nhất 1 loại request cần log, ví dụ `Untrusted Bot`, `Malicious Bot`, `Suspicious Browser`

Bấm vào `Create` để tạo log profile.
