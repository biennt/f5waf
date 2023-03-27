# F5 BIG-IP API và các công cụ tự động hoá
## F5 BIG-IP iControlREST
Tài liệu đầy đủ về F5 BIG-IP iControlREST có tại đây: https://clouddocs.f5.com/api/icontrol-rest/. Hướng dẫn này chỉ đưa ra vài ví dụ để tham khảo. 

iControl Rest sử dụng :

- Endpoint tương tự như giao diện quản trị HTTPS của F5 BIG-IP, ví dụ: `https://192.168.31.12:8443` 
- Liên quan đến các tác vụ cấu hình, quản trị thường bắt đầu bởi `/mgmt`.
- Cơ chế xác thực API sử dụng HTTP Basic Authentication với tài khoản quản trị F5 BIG-IP
- Trong nhiều trường hợp, cần tắt tính năng kiểm tra tính hợp lệ của certificate trên giao diện API này (self signed certificate)

Ví dụ dưới đây sử dụng thư viện `request` của Python để làm một số thao tác phổ biến:

```
import requests
from requests.auth import HTTPBasicAuth
from urllib3.exceptions import InsecureRequestWarning

# tài khoản quản trị để xác thực API, ở đây user và password đều là admin
cred = HTTPBasicAuth('admin', 'admin')

# mọi url của api sau đây sẽ bắt đầu bởi baseurl như thế này:
baseurl =  'https://192.168.31.12:8443/mgmt'

# hàm lấy các thông tin của một pool
def get_a_pool(poolname):
    requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)
    url = baseurl + "/tm/ltm/pool/" + poolname
    headers = {"Accept": "application/json"}
  
    response = requests.get(url,verify=False, auth = cred)
    print (str(response.text))

# hàm tạo một node, trong đa số các trường hợp, việc tạo pool bao gồm các member là các node mới thì hệ thống tự tạo node, 
# không cần gọi riêng hàm tạo node
def create_node(nodeip):
    requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)
    url = baseurl + "/tm/ltm/node"
    headers = {"Accept": "application/json"}
    data = {"name": nodeip, "address": nodeip }

    response = requests.post(url, json=data, verify=False, auth = cred)
    print (str(response.text))
# hàm xoá một node, nên làm khi xoá dịch vụ để dọn dẹp
def delete_node(nodeip):
    requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)
    url = baseurl + "/tm/ltm/node/" + nodeip 
    headers = {"Accept": "application/json"}

    response = requests.delete(url, verify=False, auth = cred)
    print (str(response.text))  

# hàm tạo một pool và gán một member vào pool đó cùng một cơ chế monitor, ví dụ tcp hoặc http
def create_pool_with_one_member(poolname, monitortype, nodeip, nodeport):
    requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)
    url = baseurl + "/tm/ltm/pool"
    headers = {"Accept": "application/json"}
    member = { "name": nodeip + ":" + nodeport, "address": nodeip }
    data = {"name": poolname, "monitor": monitortype, "members": [ member ]}
    
    response = requests.post(url, json=data, verify=False, auth = cred)
    print (str(response.text))

# hàm tạo một virtual server dạng http, và gán pool mặc định cho virtual server này
# vsname là tên virtual server
# vsaddress là địa chỉ virtual IP
# vsport là port dịch vụ được sử dụng cùng với VIP
# poolname là pool đã tạo từ trước để gán vào virtual server này
def create_virtualserver (vsname, vsaddress, vsport, poolname):
    requests.packages.urllib3.disable_warnings(category=InsecureRequestWarning)
    url = baseurl + "/tm/ltm/virtual"
    headers = {"Accept": "application/json"}
    data = { \
        "name": vsname, \
        "destination": vsaddress + ":" + vsport, \
        "mask": "255.255.255.255", \
        "pool": poolname, \
        "profilesReference": {"items": [ {"context": "all", "name": "http"}, {"context": "all", "name": "tcp"} ] }, \
        "source-address-translation": { "type": "automap"} \
        }
    print(data)
    response = requests.post(url, json=data, verify=False, auth = cred)
    print (str(response.text))


# ví dụ tạo 1 pool và virtual server như sau
#
print("create a pool with one member")
create_pool_with_one_member("testpool", "/Common/tcp", "192.168.31.11", "80")
print("create a virtual server and assign default pool")
create_virtualserver("vs_test", "192.168.31.13", "80", "testpool")
```

## F5 BIG-IP Application Services 3 Extension (AS3)

Tham khảo chi tiết về AS3 có tại đây:
- https://github.com/F5Networks/f5-appsvcs-extension. Hướng dẫn này cũng chỉ đưa ra một ví dụ đơn giản.
- https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/about-as3.html

F5 BIG-IP AS3 không có sẵn trong phần mềm BIG-IP, cần cài đặt thêm, xem hướng dẫn cài đặt tại đây: https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/installation.html#installation

Để kiểm tra hệ thống đã cài đặt thành công AS3 chưa, gọi API tới (kèm theo cơ chế xác thực basic authentication tương tự iControlREST ở trên):

`GET https://<địa chỉ quản trị BIG-IP>/mgmt/shared/appsvcs/info`

Thông tin nhận được sẽ chưa version của các thành phần trong AS3, ví dụ:
```
{
  "version": "3.43.0",
  "release": "2",
  "schemaCurrent": "3.43.0",
  "schemaMinimum": "3.0.0"
}
```

Trước hết, cần phân biệt với iControlREST vốn hoạt động dựa trên phong cách imperative, sử dụng các hành động CRUD (Create/Read/Update/Delete), nghĩa là tác động lên từng 'resource'. Lấy ngữ cảnh là 1 virtual server, để tạo được nó bằng iControlREST, cần thực hiện lần lượt các lời gọi API như:

- Tạo một danh sách các node gồm node name và các địa chỉ IP tương ứng
- Tạo một pool chứa các node đã tạo ở bước trên, gán vào thuật toán cân bằng tải và cơ chế monitor
- Tạo một virtual server trỏ vào pool đã tạo ở bước trên, gán vào VIP, port, các profile tương ứng với giao thức và các thuộc tính cần thiết khác

Như thế, cần 3 lời gọi API để hoàn thành nhiệm vụ.

Đối với AS3, hoạt động theo phong cách declarative, chỉ cần 1 lời gọi API duy nhất, chứa đầy đủ các thông tin cần thiết để tạo 1 virtual server là được.

Ví dụ, gọi API tới endpoint như bên dưới kèm theo thông tin xác thực

```

POST https://192.168.31.12:8443/mgmt/shared/appsvcs/declare
Authorization: Basic YWRtaW46YWRtaW4=
```

Nội dung của payload như sau:
```
{
  "class": "AS3",
  "action": "deploy",
  "persist": true,
  "declaration": {
    "class": "ADC",
    "schemaVersion": "3.0.0",
    "label": "SampleLabel",
    "PartitionName": {
      "class": "Tenant",
      "vsName": {
        "class": "Application",
        "vsName": {
          "class": "Service_HTTP",
          "virtualAddresses": [
            "192.168.31.13"
          ],
          "pool": "testpool"
        },
        "testpool": {
          "class": "Pool",
          "monitors": [
            "tcp"
          ],
          "members": [{
            "servicePort": 80,
            "serverAddresses": [
              "192.168.31.11"
            ]
          }]
        }
      }
    }
  }
}

```

## F5 Ansible Modules








