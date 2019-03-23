## Secrets
Với một số loại dữ liệu như mật khẩu, token, private encryption keys,... cần được giữ an toán. K8s cung cấp Secrets. Nó cóc nhiệp vụ phân phối những dữ bí mật chỉ cho những node cần truy cập bí mật. Tuy vậy, ở các nodes, secrets luôn luôn lưu trữ trong ram và không bao giờ ghi vào đĩa. Trên Master node, secrets được lưu trữ trong form mã hóa bên trong etcd database.

Secrets và Config Maps có thể được chuyển đến các container bằng cách mount một volume hoặc thông qua biến môi trường. Cho ví dụ, file _mysql-secret.yaml_ được định nghĩa là một thông tin đăng nhập của người dùng.
```sh
apiVersion: v1
kind: Secret
metadata:
  name: mysql
  namespace:
data:
  MYSQL_RANDOM_ROOT_PASSWORD: eWVz
  MYSQL_DATABASE: ZW1wbG95ZWVz
  MYSQL_USER: YWRtaW4=
  MYSQL_PASSWORD: cGFzc3dvcmQ=
```

Dữ liệu mật sẽ được mã hóa base64 bằng cách:
```sh
$ echo -n "admin" | base64
YWRtaW4=
$ echo -n "password" | base64
cGFzc3dvcmQ=
...
```

Dữ liệu có thể được thông qua biến môi trường của MySQL server. Chúng được định nghĩa trong file _mysql-pod-secret.yaml_
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
    envFrom:
    - secretRef:
        name: mysql
    ports:
    - name: mysql
      protocol: TCP
      containerPort: 3306
```


