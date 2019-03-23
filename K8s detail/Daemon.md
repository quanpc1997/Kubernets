## Daemons
Một Daemon Set là một loại controller có nhiệm vụ bảo đảm mỗi node trong một cluster chạy một pod. Khi node mới được thêm vào cluster, một pod mới sẽ được thêm vào node. Khi node bị xóa đi trong cluster nào đó, pod đang chạy trong nó cũng bị xóa và không scheduled được node khác. Xóa một Daemon Set sẽ làm sạch tất cả các pod nó khởi tạo.

Sau đây là file cấu hình _nginx-daemon-set.yaml_ định nghĩa một daemon set cho ứng dụng nginx:
```sh
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    run: nginx
  name: nginx-ds
  namespace:
spec:
  selector:
    matchLabels:
      run: nginx-ds
  template:
    metadata:
      labels:
        run: nginx-ds
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
```

Khởi tạo một daemon set và lấy thông tin id từ nó:
```sh
kubectl create -f _nginx-daemon-set.yaml

kubectl get ds nginx-ds -o wide
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-ds   2         2         2       2            2           <none>          25s   nginx        nginx:latest   run=nginx-ds
```
Có chính xác 2 pods vì chúng ta có 2 node. Và mỗi một pod đang chạy trong một pod khác nhau:
```sh
kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-ds-b9rnc           1/1       Running   0          7m        10.38.0.20   node1
nginx-ds-k5898           1/1       Running   0          7m        10.38.1.25   node2
```

Thử xóa một node từ cluster, pod đang chạy cũng bị xóa và không scheduled trên những node khác Điều mà vẫn xảy ra trên các loại controller khác.
```sh
kubectl delete node node2

kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-ds-b9rnc           1/1       Running   0          7m        10.38.0.20   node1
```
Thêm trở lại node vào cluster, chúng ta có thể xem một pod mới được lên lịch trong node.
```sh
kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-ds-b9rnc           1/1       Running   0          7m        10.38.0.20   node1
nginx-ds-61swl           1/1       Running   0          3s        10.38.2.18   node2
```

Để xóa một daemon set
```sh
kubectl delete ds nginx-ds
```

Một daemon set object không bị quản lí bởi K8s scheduler vì chỉ có một pod cho mỗi node. Để kiểm tra, đăng nhập vào node master và stop the scheduler.
```sh
kubectl stop kube-scheduler
```

Bây giờ khởi tạo daemon set.
```sh
kubectl create -f nginx-daemon-set.yaml
```

Chúng ta cần xem những pods scheduled trên mỗi node
```sh
kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-ds-b9rnc           1/1       Running   0          7m        10.38.0.20   node1
nginx-ds-61swl           1/1       Running   0          3s        10.38.2.18   node2
```

Ngoài ra chúng ta cũng có thể chỉ set trên một node nào đó bằng cách
```sh
...
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
...
      nodeSelector:
        kubernetes.io/hostname: node3

```

