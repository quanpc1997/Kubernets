# Khái niệm cơ bản

Tham khảo: http://tutorial.kubernetes.noverit.com/

_(Nhấp vào tiêu đề để đọc chi tiết thêm)_
## Một số khái niệm cơ bản.

### Pod
	* Là một nhóm các container chạy một số ứng dụng, chia sẻ tài nguyên và có cùng một địa chỉ IP.
	* Pod cung cấp 2 loại tài nguyên là Network và Storage.

### Labels và Annotations
* **Labels**: 
	* Gán nhãn vào một đối tượng _(pod, node)_.
	* Nằm ngay dưới _metadata_.
	* Là định danh để filter hoặc để áp dụng 1 service.
* **Annotation**: là chú thích.

[Chi tiết](../K8s detail/Label and Annotations.md)

### Replica Set
	* Quy định số bản pod giống hệt nhau chạy cùng 1 số dịch vụ.
	* Khi có bất kì vấn đề nào khiến 1 số pod không chạy được. Nó sẽ tự tạo thêm pod mới đúng bằng số pod bị chết.
	* Thông qua Replica Set controller, K8s sẽ quản lý vòng đời của các Pod, scale up/down, deployment, monitoring,...
	* Để config một Replica set bao gồm: resplicas, selector, template
[Chi tiết](../K8s detail/ReplicaSets.md)

### Deployment
	* Là một phiên bản cao hơn của replicas controller. Quản lí cách thức triển khai của replicas set. Chúng có khả năng cập nhật phiên bản mới cho các replica set và images, khả năng rollback lại phiên bản cũ mà không gây ảnh hướng đến công việc các pod đang chạy.
	* Nếu trong quá trình deployment mà bị lỗi ở pod đầu tiên thì toàn bộ việc deployment sẽ bị treo. Việc cần làm duy nhất sau đó là rollback lại.
[Chi tiết](../K8s detail/Deployment.md)

### Services
	* Là một liên kết logic giữa các pods.
	* Cung cấp IP, DNS cho các pods mà có gán nhãn trùng với nhãn của nó. 
	* Dễ dàng triển khai quản lí và cân bằng tải, giúp scale một cách dễ dàng.
	* Đây là nơi bạn có thể định cấu hình cân bằng tải cho nhiều pod và expose các pod đó.
[Chi tiết](../K8s detail/Services.md)

### Volumes
	* Là nơi lưu trữ dữ liệu trong các container trong pod.
	* Có rất nhiều loại Volumes nhưng ta chỉ tập chung vào 2 loại chính là:
		* **emptyDir**: Chỉ tồn tại khi pods còn tồn tại. Nó được lưu trực tiếp trên đĩa hoặc trên ram.
			**=> Không phù hợp để lưu dữ liệu lâu dài** 
		* **Host Path Volume**: Giống như mount trong linux. Khi pod bị xóa thì dữ liệu vẫn còn. Dễ dàng sao lưu và backup.
[Chi tiết](../K8s detail/Volumes.md)

### ConfigMap
	* Là một file config. Giúp cập nhật sửa chữa các config từ bên ngoài và bên trong.
	* Có 2 cách config:
		* Mount Config Map as volume:
		* Pass Config Map by environment variables:

	* Thường được dùng để làm việc với những dữ liệu không nhạy cảm.
[Chi tiết](../K8s detail/ConfigMap.md)

### Secrets
	* Là một file config giống như configmap nhưng dùng để làm việc với dữ liệu nhạy cảm như mật khẩu, token, thông tin,...
	* Dữ liệu sẽ được mã hóa Base64
	* Config giống hệt configmap
[Chi tiết](../K8s detail/Sercert.md)

### Deamons
	* Một Daemon Set là một loại controller có nhiệm vụ bảo đảm mỗi node trong một cluster chạy một pod. Khi node mới được thêm vào cluster, một pod mới sẽ được thêm vào node. Khi node bị xóa đi trong cluster nào đó, pod đang chạy trong nó cũng bị xóa và không scheduled được node khác. Xóa một Daemon Set sẽ làm sạch tất cả các pod nó khởi tạo.
[Chi tiết](../K8s detail/Daemon.md)

### Jobs
	* Được khởi tạo để làm một số công việc _(chạy một số lệnh)_ khi được khởi tạo. 
	* Một Jobs khởi tạo 1 hay nhiều pods và đảm bảo rằng một trong số chúng đã thành công. Khi tất cả pod thành công, thì job hoàn thành.
[Chi tiết](../K8s detail/Job.md)

### CronJob
	* Giống như Job nhưng bị quản lí thời gian. Bó chỉ chạy theo định kì nhất định được lên lịch sẵn.
	* Thời gian được đánh theo kiểu crontab
[Chi tiết](../K8s detail/CronJob.md)

### Namespace
	* Là định danh một nhóm cluster ảo và cùng được áp đặt một số thuộc tính và tính chất nào đó. ví dụ như cùng chạy loại service, ...
	* Có 3 loại namespace cần quan tâm:
		* **default**: Là namespace mặc định cho các object.
		* **kube-system**: namespace cho những đối tượng khởi tạo bởi K8s system.
		* **kube-public**: namespace được khởi tạo tự động và có thể đọc bởi tất cả người dùng(bao gồm những người không được chứng thực). Namespace này chủ yếu được dành riêng cho việc sử dụng cluster, trong trường hợp một số tài nguyên sẽ được hiển thị và có thể được công khai trên toàn bộ cụm.

	* Không phải đối tượng nào được tạo ra trong K8s cũng bắt buộc phải thuộc 1 namespace nào đó.
	* Xóa một namespace tức là sẽ xóa đi tất cả những cài đặt bên trong các object đó.
[Chi tiết](../K8s detail/Namespace.md)

### Quotas and Limits
	* Đặt giới hạn ngạch cho việc sử dụng, dùng chung tài nguyên. Nó có thể giới hạn số tài nguyên được sử dụng cũng như quá trình tạo pod sao cho không vượt quá hạn ngạch đưa ra.	
[Chi tiết](../K8s detail/Quotas and Limits.md)

