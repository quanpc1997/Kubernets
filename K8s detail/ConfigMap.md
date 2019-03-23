## Config Maps
Kubernetes cho phép tách các tùy chọn cấu hình thành một đối tượng riêng biệt được gọi là ConfigMap. Đây là bản đồ chứa các cặp key/value với các giá trị short literals to full config files. Một ứng dụng không cần để đọc trực tiếp ConfigMap hoặc trậm trí biết rằng nó tồn tại. Nội dung của map được chuyển vào trong container dưới dạng biến môi trường hoặc files trong volumes.

### Mount Config Map 
Config Map mounted như một volume vào bên trong pod. Cho ví dụ, khởi tạo một ConfigMap từ file cấu hình _nginx.conf_
```sh
kubectl create configmap nginx --from-file=nginx.conf
```
Định nghĩa một nginx pod mouting vào config map. Tạo file _nginx-pod-cm.yaml_ và tạo pod:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace:
  labels:
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d
        readOnly: true
  volumes:
    - name: config
      configMap:
        name: nginx
```

Kiểm tra pod vừa tạo đã được mount đến Config Map chưa như file cấu hình mặc định:
```sh
kubectl exec nginx cat /etc/nginx/conf.d/nginx.conf
```

Một cách khác thay thế. Ta tạo file nginx-conf.yaml
```sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx
  namespace:
data:
  nginx.conf: |
    server {
        listen       80;
        server_name  www.noverit.com;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
```
Sử dụng Config Map và hiển thị thông qua một volume giúp cập nhật cấu hình mà không phải tạo lại pod hoặc thậm chí khởi động lại container. Khi cập nhật Config Map, các tệp trong tất cả các volumes tham chiếu nó được cập nhật. Khi hệ thống phát hiện ra có sự thay đổi, hệ thống sẽ tải lại chúng.

Cho ví dụ về cập nhật và cấu hình ở trên. Edit Config Map
```sh
kubectl edit cm nginx
```

Sau đó kiểm tra ứng dụng nginx đang chạy bên trong pod đã tải lại cấu hình.
```sh
kubectl exec nginx cat /etc/nginx/conf.d/nginx.conf
```

Trong ví dụ trước, chúng ta đã mount volume như một directory, Điều đó có nghĩa là bất kì file nào mà lưu trữ bên trong _/etc/nginx/conf.d_ bên trong Conatainer image sẽ bị ẩn. Đây là điểm chung - cái mà xảy ra trong Linux khi mà mount một filesystem vào trong một directory và directory sau đó chỉ chwuas những file từ filesystem đã mount. trong khi các tập tin gốc không thể truy cập được.

Trong K8s nó có thể chỉ mount các individual files từ Config Map vào thư mục hiện có mà không ẩn các tệp hiện có được lưu trữ trên thư mục. Tham khảo đến ví dụ trước, Directory _/etc/nginx/conf.d/_ của container filesystem, đã chứa đựng một file cấu hình mặt định gọi là _default.conf_. Chúng ta không muốn ẩn file này nhưng chúng ta chỉ muốn thêm một file cấu hình cọi là _custom.conf_ cho nginx container.

Khởi tạo một Config Map bằng cách tạo file mô tả _nginx-custom.conf_ sau:
```sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx
  namespace:
data:
  custom.conf: |
    server {
        listen       8080;
        server_name  www.noverit.com;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
``` 

và sau đó tạo một nginx pod từ file mô tả sau _nginx-pod-cm-custom.yaml_:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace:
  labels:
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 8080
    volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d/custom.conf
        subPath: custom.conf
        readOnly: true
  volumes:
    - name: config
      configMap:
        name: nginx
```

Kết quả là sẽ mount cả 2 tệp cấu hình là custom và default:
```sh
kubectl exec nginx -- ls -lrt /etc/nginx/conf.d/
total 8
-rw-r--r-- 1 root root 1093 Sep 25 15:04 default.conf
-rw-r--r-- 1 root root  166 Sep 27 11:45 custom.conf
```

**Chú ý rằng khi mounting một single file trong một container và Config Map thay đổi thì file sẽ không được update**

### Pass Config Map by environment variables
ngoài ra để mount một volume, configuration values contained bên trong một Config Map có di chuyển trực tiếp đến một container bên trong biến môi trường. Cho ví dụ, sau đây file cấu hình _mysql-cmk.yaml_ định nghĩa một Config Map container cấu hình thông số cho MySQL application.

```sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  namespace:
data:
  MYSQL_RANDOM_ROOT_PASSWORD: "yes"
  MYSQL_DATABASE: "employees"
  MYSQL_USER: "admin"
  MYSQL_PASSWORD: "password"
```

Sau đây _mysql-pod-cmk.yaml_ file định nghĩa MySQL pod sử dụng giá trị từ map bên trên.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  namespace:
  labels:
    run: mysql
spec:
  containers:
  - name: mysql
    image: mysql:5.6
    env:
    - name: MYSQL_RANDOM_ROOT_PASSWORD
      # The generated root password will be printed to stdout
      # kubectl logs mysql | grep GENERATED
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: random_root_password
    - name: MYSQL_DATABASE
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: database
    - name: MYSQL_USER
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: user
    - name: MYSQL_PASSWORD
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: password
    ports:
    - name: mysql
      protocol: TCP
      containerPort: 3306
```

<Còn nữa>

