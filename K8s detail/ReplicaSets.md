## Replica Sets
Replica Set là thông số xác định số bản Pod giống hệt nhau trong bất kì thời gian. Phụ thuộc vào công việc mà ta có thể set số pod. Nếu có Pod nào bị lỗi thì chúng sẽ tự thay thế bằng cách tạo mới một Pod.
Một Replica Set configuration bao gồm:
  * resplicas: <Số bản sao>
  * selector: 
  * template:

Selector là những label được gán cho các pod và bị quản lý bởi replica set. Label nằm bên trong phần định nghĩa của Pod mà replica set. replica set được sử dụng để chọn lựa nhằm mục đích kiểm soát có bao nhiêu Instances của Pod đang chạy để điều chỉnh khi cần thiết.

Ta xem xét ví dụ sau:
```sh
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    run: nginx                              // Label của của giá trị kind
  namespace:
  name: nginx-rs                            // Tên của ReplicaSet
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx                            // Label này trùng với label của Kind 
  template:
    metadata:
      labels:
        run: nginx                          // Label của Pod 
    spec:
      containers:
      - image: nginx:1.12
        imagePullPolicy: Always
        name: nginx                         // Tên của containers
        ports:
        - containerPort: 80
          protocol: TCP
      restartPolicy: Always
```
**Chú ý: Với mỗi loại kind thì có một loại apiVersion khác nhau. Phiên bản này sẽ thường update tên thường xuyên. Cần gg để tìm hiểu cho kĩ để điền thông số phù hợp.**

Thống kê và mô tả một replica set:
```sh
kubectl get rs
NAME      DESIRED   CURRENT   READY     AGE
nginx     3         3         3         1m

kubectl describe rs nginx
Name:           nginx
Namespace:      default
Image(s):       nginx:latest
Selector:       run=nginx
Labels:         run=nginx
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
```

Replica Set làm cho hệ thống scale một cách dễ dàng hơn. Việc này có thể làm bằng tay hoặc để hệ thống tự động.

Ví dụ: Scale down xuống 0 replicas để xóa hết tất cả các pods.
```sh
kubectl scale rs nginx --replicas=0

kubectl get rs nginx
NAME      DESIRED   CURRENT   READY     AGE
nginx     0         0         0         7m
```

Sau đó Scale out lên để khởi tạo những pod mới.
```sh
kubectl scale rs nginx --replicas=9

kubectl get rs nginx
NAME      DESIRED   CURRENT   READY     AGE
nginx     9         9         0         9m
```

Trong trường hợp bị lỗi tại một node nào đó, replica set sẽ giữ nguyên số nhóm bằng cách lập lịch cho các container đang chạy trên node hỏng để di chuyển đến các nút trong cụm còn lại.

Để xóa một replica set:
```sh
kubectl delete rs/nginx
```

Xóa tất cả các replica set:
```sh
kubectl delete rs/nginx --cascade=false
```
Bây giờ không có gì đang quản lí pods, nhưng chúng ta có thể luôn luôn khởi tạo những replication set cùng với proper label selector và tạo quản lí chúng trở lại.

Một bản sao replica set duy nhất có thể match những pod cùng với các label _env=production_ và các nhóm có label _env=dev_ cùng một lúc. Chúng ta cũng có thể match tất cả pods vào bên trong một label cùng với key _run_. 
```sh
...
selector:
   matchExpressions:                
     - key: run
       operator: In
       values:
         - nginx
           web
```

