## Labels
Trong K8s, Lable là một hệ thống để tổ chức các đối tượng vào bên trong các nhóm. Labels hoạt động dạng Key-Value để gắn vào mỗi một đốt tượng.

Để thêm một label vào một pod. Ta thêm chúng vào dưới metadata bên trong phần định nghĩa pod:
```sh
apiVersion: v1
kind: Pod 
metadata:
	labels: 
		run: nginx
...
```

Để thêm label một pod đang chạy
```sh
kubectl label pods nginx type=webserver 
```

Để lấy danh sách pod trong label
```sh
kubectl get pods -l type=webserver

NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          21m
```

Lable không chỉ gán nhãn vào các pod mà còn có thể làm được cả với Worker Nodes base.
```sh
kubectl label node kuben01 rack=rack01

kubectl get nodes -l rack=rack01

NAME      STATUS    AGE
kuben01   Ready     2d
  
kubectl label node kuben02 rack=rack01
  
kubectl get nodes -l rack=rack01
  
NAME      STATUS    AGE
kuben01   Ready     2d
kuben02   Ready     2d
```

## Annotations
Giống như Label, nó cũng có cơ chế Key-Value. Nhưng nó không thể sử dụng để gán vào các nhóm như lable. Trong khi ta có thể lựa chọn thông qua các selectors. Annotations có thể làm mở rộng ý nghĩa cho label. Một số chú thích được Kubernetes tự động thêm vào các đối tượng, nhưng những chú thích khác có thể được thêm bởi người dùng.

Để thêm một Annotations vào một pod. Ta thêm chúng vào dưới metadata bên trong phần định nghĩa pod:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx          
  annotations:
    readme: "before to run this pod, make sure you have a service account defined."
  namespace:
  labels:
    run: nginx
...
```

