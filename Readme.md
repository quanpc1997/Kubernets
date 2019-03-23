# Kubernetes

**Tham khảo**: https://blog.vietnamlab.vn/2018/08/08/nhap-mon-kubernetes/

## Kubernetes là gì?
Là một nền tảng mã nguồn mở giúp quản lý, vận hành nhiều container trên nhiều host một cách hiệu quả.

## Tại sao cần dùng Kubernetes?
* Do khả năng cho phép mở rộng các container một cách mạnh mẽ, triển khai nhiều container trên nhiều server host. 
* Khả năng quản lý nhiều container một cách dễ dàng(Quản lí tình trạng container theo thời gian thực).
    - Khả năng quản lý: Gộp nhóm các container với nhau thành những "Pod" để tiện lên lịch, giảm sát hoạt động
* Kubernetes cần tích hợp với networking, storage, security, telemary và các dịch vụ khác để cung cấp một cơ sở hạ tầng container toàn diện.
* Có khả năng kết hợp với nhiều công nghệ khác như Atomic Registry, Open vSwitch, heapster, OAuth và SELinux,... giúp hoàn thiện tất cả các thành phần của cơ sở hạ tầng container của mình. 

## Công dụng của Kubernetes

Ta có thể làm nhiều công việc bằng cách sử dụng Kubernetes:

* Việc quản lý hàng loạt docket host
* Container Scheduling: 
    - Tùy vào trạng thái tài nguyên hệ thống nhiều hay ít, nghỉ ngơi hay quá tải của các host mà tự động phân bỏ quản lý tài nguyên hay phân bổ cho các host khác để giảm quá tải.
* Rolling update
* Scaling/Auto Scaling: 
    - Tự động tạo ra các container giống nhau làm cho hệ thống có sức chịu lỗi cao. Tự động load balancing.
* Self-hearing trong trường hợp có lỗi xãy ra. (Có khả năng phát hiện và tự correct lỗi)
    - giả sử 1 container bị xảy ra vấn đề(bị chết hoặc bị xóa), cơ chết self-hearing sẽ tự động phục hồi 
* Service discovery
* Load balancing: Điều phối truy cập.
* Quản lý data
* Quản lý work node
* Quản lý log
* Infrastructure as Code

Dựa vào các công nghệ khác để hoàn thiện:
- Jenkins: Deploy container đến Kubernetes

- OpenStack: Cấu trúc k8s liên kết với Cloud

- Prometheus: Monitor Kubernetes.

- Registry, thông qua các dự án như Atomic Registry hoặc Docker Registry.

- Networking, thông qua các dự án như OpenvSwitch và định tuyến cạnh thông minh (intelligent edge routing).

- Telemetry, thông qua các dự án như heapster, kibana, hawkular và elastic.

- Security, thông qua các dự án như LDAP, SELinux, RBAC và OAUTH với multi-tenancy layers.

- Automation, với việc bổ sung các Playbook Ansible để cài đặt và quản lý vòng đời của cụm.

- Services, thông qua một danh mục phong phú về nội dung được xử lý trước của các mẫu ứng dụng phổ biến.

## Docker và Kubernetes
Kubernetes lên lịch cho một pod vào một node. Kubelet liên tập thu nhập các trạng thái của container. Khi có lệnh từ quản trị viên thì Kubernetes sẽ ra lệnh cho docker thực hiện.
Sử dụng Kubernetes giúp cho quản trị viên dễ dàng quản lý các container một cách tự động thay vì phải quản lí từng container bằng tay.


# Kiến trúc và các thành phần trong K8S

## Kiến trúc
Kiến trúc của Katanetes gồm có 2 thành phần chính:
    * **Master node**: Là thành phần điều khiển toàn bộ các hoạt dộng chung và kiểm soát các container trên Worker Node. Các thành phần chính trên master node gồm: _API-server, Controller-manager, Schedule, Etcd và cả Docker Engine_.
    * **Worker node**: là môi trường để chạy các container. Thành phần chính của nó bao gồm _kubelet, kube-proxy_ và cả _Docker_.
    

Thông thường thì số lượng **Master node** sẽ ít hơn node worker. Nhưng nếu hệ thống cần _High Availability_ thì cần triển khai nhiều **Master Node**.

###  Namespace 
Đây là một công cụ dùng để nhóm hoặc tách các nhóm đối tượng. Namespaces được sử dụng để kiểm soát truy cập, kiểm soát truy cập network, quản lý resource và quoting.

### Etcd 
* Là một thành phần database phân tán nằm trong node master và hoạt động theo cơ chế key/value. Nó có nhiệm vụ lưu trữ tất cả thông tin trong Cluster. 
* Etcd sử dụng port 2380 để lắng nghe và port 2379 để gửi request.
* Ectd nằm trên node master.

### API-server
* API server là thành phần trung gian tiếp nhận yêu cầu của người dùng.
* Hoạt động trên cổng 6443(HTTPS) và 8080(HTTP)
* API-server nằm trên node master..

### Controller-manager
* Là thành phần có nhiệm vụ quản lí, xử lí các tác cụ trong cluster. Chúng gồm:
    - Node Controller: Tiếp nhận và trả lời các thông báo khi có 1 node bị chết.
    - Replication Controller: Đảm bảo các công việc duy trì chính xác số lượng container cùng thực hiện một ứng dụng.
    - Endpoints Controller: 
    - Service Account và Token Controller: Quản lí các Account và token để có quyền sử dụng các API.
* Controller-manager chạy trên cồng 10252 và được cài đặt trên Master Node.

### Schedule
* Kube-Schedule có nhiệm vụ giám sát việc sử dụng tài nguyên mỗi máy để đảm bảo hệ thống không bị quá tải. Mỗi khi có yêu cầu tạo pod. Nó lựa chọn các node sao cho phù hợp nhất dựa vào cơ chế lập lịch.
* Chạy trên cổng 10251.

### Kubelet
* Là một thành phần chính chạy trong các node worker. Khi kube-Schedule đã xác định được một pod được chạy trên node nào thì nó sẽ gửi thông tin cấu hình (images, volume,...) tới lubelet trên node đó. Dựa vào thông tin nhận được thì kubelet sẽ tiến hành tạo container theo yêu cầu.
* Vai trò chính của kubelet:
    - theo dõi các các pod trên node được gán đề hoạt động.
    - cung cấp các volume cho pod.
    - Quản lí các container của pod và báo cáo trạng thái về pod để cụm cluster biết được các container có hoạt động ổn không.

* Kubelet chạy trên các node worker và sử dụng port 10250, 10255.

##3 Proxy
* Là thành phần giao tiếp giữa mạng trong và mạng ngoài.
* Kube-proxy được cài đặt trên node worker và sử dụng port 31080.

### CLI
*kubectl là thành phần cung cấp câu lệnh để người dùng tương tác với K8S. kubectl có thể chạy trên bất cứ máy nào có kết nối với K8S API-server.

## Pod network
Đây là một cụm nhằm đảm bảo các container có thể truyền thông được với nhau. Có nhiều lựa chọn về pod network.

