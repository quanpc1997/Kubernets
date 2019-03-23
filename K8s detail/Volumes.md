## Volumes
File system của container chỉ sống khi container còn sống. Nếu có bất cứ vấn đề gì thì ta cần có cơ chế lưu trữ liên tục. Một K8s Volume có một thời gian sống rõ ràng, giống như pod của nó. Dữ liệu được lưu trữ trong một K8s volume tồn tại trong quá trình di chuyển, khởi động lại, crash của hosting pod.

Trong phần này, chúng ta sẽ khởi tạo 2 kiểu volume khác nhau:
	* **emptyDir**
	* **hostPath**

tuy vậy, K8s hỗ hợ nhiều hơn 2 loại phân vùng trên. Hãy đọc trong tài liệu chính thức để biết thêm chi tiết:

### Empty Dir Volume
Tại đây có một ví dụ đơn giản. Ta tạo file _nginx-volume.yaml_:
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
    volumeMounts:
    - name: content-data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: content-data
    emptyDir: {}
```

File cấu hình nginx pod là _/var/share/nginx/html_ được mounted như volume. Kiểu volume này là **emptyDir**. Kiểu volume này được khởi tạo khi mà pod được gán cho một node và nó tồn tại miễn là pod đang chạy trong node. Tên của nó nói lên tất cả, nó được khởi tạo rỗng. Khi pod bị xóa đi từ node do bất cứ lí do gì, thì dữ liệu bên trong **emplyDir volume*** cũng sẽ bị xóa. 

**Empty Dir Volume** được lưu trữ tại _/var/lib/kubelet/pods/POD_UID/volumes/kubernetes.io~empty-dir/_ của worker node nơi mà pode đang chạy. 

**Chú ý rằng một container crash nhưng pod không bị xóa, thì dữ liệu trong emptyDir volume an toàn khi container bị crash**

Kiểm tra pod IP cho worker node hosting:
```sh
kubectl get po/nginx -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP            NODE
nginx     1/1       Running   0          13m       10.38.3.167   kubew03
```

Thay vì lưu trữ trên đĩa ta có thể đặt dữ liệu trên **tmpfs filesystem**, tức là trong bộ nhớ ram thay vì trên đĩa. Để thực hiện việc này ta làm như sau:

```sh
...
volumes:
  - name: html
emptyDir:
      medium: Memory
...
```

### Host Path Volume
Một kiểu volume khác chúng ta sẽ sử dụng là _hostPath_. Với kiểu volume này, volume được mount từ một directory tồn tài trong file system của node hosting the pod. Dữ liệu bên trong directory được an toàn kể cả khi container bị xóa pod. Tuy vậy, nếu pod bị di chuyển từ node này sang node khác, dữ liệu cũ sẽ không thể truy cập được trong node mới.

Ví dụ cơ bản, định nghĩa một nginx pod sử dụng thư mục _/mnt_ như là data volume cho html content. Tạo file _nginx-host-volume.yaml_
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
    volumeMounts:
    - name: content-data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: content-data
    hostPath:
      path: /mnt
```
Chú ý rằng, thư mục _/mnt_ phải đại diện trong một node hosting trước khi pod được khởi tạo.

Kiểm tra lại Ip pod:
```sh
kubectl get po/nginx -o yaml | grep podIP
podIP: 172.30.41.7

echo "Hello from $(hostname)"  > /mnt/index.html
```

```sh
curl 172.30.41.7:80
Hello from quanpc
```

Dữ liệu trong host dir volume sẽ được tồn tại với bất kì vấn đề gì. Để kiểm tra ta hãy xóa pod và khởi tạo trở lại.
```sh
kubectl delete pod nginx
pod "nginx" deleted

kubectl create -f nginx-host-volume.yaml
pod "nginx" created

kubectl get po/nginx -o yaml | grep podIP
  podIP: 172.30.41.7

curl 172.30.41.7:80
Hello from quanpc
``` 
Điều này chỉ khả dụng khi kubernetes lên lịch cho nginx pod trên cùng một nút worker như trước đây.

