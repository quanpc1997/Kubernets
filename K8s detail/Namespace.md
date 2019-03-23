## Namespace
K8s hỗ trợ nhiều cluster ảo bởi cùng một cụm vật lý. Những cluster ảo được gọi là namespace. Tên của Namespace là duy nhất. Các Object khác nhau trong những không gian tên khác nhau có thể cùng tên.

Namespace cung cấp một 

Có 3 loại namespace:
  * **default**: Là namespace mặc định cho các object.
  * **kube-system**: namespace cho những đối tượng khởi tạo bởi K8s system.
  * **kube-public**: namespace được khởi tạo tự động và có thể đọc bởi tất cả người dùng(bao gồm những người không được chứng thực). Namespace này chủ yếu được dành riêng cho việc sử dụng cluster, trong trường hợp một số tài nguyên sẽ được hiển thị và có thể được công khai trên toàn bộ cụm.

Để xem namespaces:
```sh
kubectl get namespaces
NAME          STATUS    AGE
default       Active    7d
kube-system   Active    7d
```

Để xem những đối tượng thuộc namespace:
```sh
kubectl get all --namespace default

NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/nginx   2         2         2            2           52s
NAME             CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
svc/kubernetes   10.254.0.1     <none>        443/TCP          7d
svc/nginx        10.254.33.40   <nodes>       8081:31000/TCP   51s
NAME                  DESIRED   CURRENT   READY     AGE
rs/nginx-2480045907   2         2         2         52s
NAME                        READY     STATUS    RESTARTS   AGE
po/nginx-2480045907-56t21   1/1       Running   0          52s
po/nginx-2480045907-8n2t5   1/1       Running   0          52s
```

Để xem tất cả đối tượng thuộc tất cả các namespace
```sh
kubectl get service --all-namespaces
NAMESPACE     NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
default       kubernetes             10.254.0.1       <none>        443/TCP          7d
default       nginx                  10.254.33.40     <nodes>       8081:31000/TCP   2m
kube-system   kube-dns               10.254.3.100     <none>        53/UDP,53/TCP    3d
kube-system   kubernetes-dashboard   10.254.180.188   <none>        80/TCP           1d
```

Chú ý: Không phải K8s nào cũng phải thuộc namespace. 

Xác định một Project namespace mới từ file cấu hình _projectone-ns.yaml_:
```sh
apiVersion: v1
kind: Namespace
metadata:
  name: projectone
```

Tạo một namespace mới:
```sh
kubectl create -f projectone-ns.yaml
namespace "projectone" created

kubectl get ns project-one
NAME          STATUS    AGE
projectone   Active    6s
```

Những Object có thể được gán vào một namespace cụ thể. Namespace luôn nằm trong metadata:
Cho ví dụ:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: projectone
  labels:
    run: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
``` 

Tạo nginx pod và kiểm tra nó chỉ sống trong project-one namespace:
```sh
kubectl create -f nginx-pod.yaml
pod "nginx" created

kubectl get pod nginx -n projectone
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          51s

kubectl get pod nginx
Error from server (NotFound): pods "nginx" not found
```

**Chú ý: Xóa một namespace tức là sẽ xóa tất cả những object bên trong namespace đó.**


