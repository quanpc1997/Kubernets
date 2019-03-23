## Services

Một Service có thể được định nghĩa như là một liên kết logic của các pods. Nó cung cấp một IP và DNS cho các pods có thể truy cập đến nó. Nó rất dễ dàng để quản lí để cấu hình cân bằng tải. Nó giúp pods mở rộng một cách dễ dàng.

Để khởi tạo một service cho nginx webserrver, ta tạo file _nginx-service.yaml_:
```sh
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  selector:
    run: nginx
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 80
  type: ClusterIP
```

Khởi tạo service:
```sh
kubectl create -f nginx-service.yaml
service "nginx" created

kubectl get service -l run=nginx
NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx     10.254.60.24   <none>        8000/TCP    38s
```

Mô tả service:
```sh
kubectl describe service nginx

Name:                   nginx
Namespace:              default
Labels:                 run=nginx
Selector:               run=nginx
Type:                   ClusterIP
IP:                     10.254.60.24
Port:                   <unset> 8000/TCP
Endpoints:              172.30.21.3:80,172.30.4.4:80,172.30.53.4:80
Session Affinity:       None
No events.
```

Pods có thể được thêm vào một cách tùy ý. Miễn rằng label _run=nginx_ được liên kết với bất kì pod mà chúng ta liên kết với dịch vụ. 

Định nghĩa một pod từ tập sau và không có bất kì nhãn nào:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```
Sau khi khởi tạo nó không được liên kết với nginx service.
```sh
kubectl get endpoints | grep nginx
NAME         ENDPOINTS                                    AGE
nginx        172.30.21.2:80,172.30.4.2:80,172.30.4.3:80   40m
```
Bây giờ ta thêm label _run=nginx_ vào pod và xem kết quả:
```sh
kubectl label pod nginx run=nginx
pod "nginx" labeled

kubectl get endpoints | grep nginx
NAME         ENDPOINTS                                                AGE
nginx        172.30.21.2:80,172.30.4.2:80,172.30.4.3:80 + 1 more...   46m
```

