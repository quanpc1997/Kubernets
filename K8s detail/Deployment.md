## Deployments
Deployments là một phiên bản cao hơn của replicas controller. Chúng quản lí cách thức triển khai của replica sets. Chúng có khả năng cập nhật các replica set, khả năng rollback lại phiên bản cũ. Đặc biệt, chúng có khả năng thay đổi triển khai dữa chừng 

Chúng cung cấp nhiều tính năng cập nhật của _matchLabels_ và _selectors_. 

### Hoạt động trong Deployment
**Updating** - Người dùng có thể update những triển khai đang diễn ra trước khi nó hoàn thành. Trong đó, việc triển khai hiện tại sẽ được giải quyết và việc triển khai mới sẽ được tạo ra.

**Deleting** - Người dùng có thể pause/stop deployment bằng cách xóa nó trước khi nó hoàn thanh. Nếu chúng ta khởi tạo lại nó sẽ giống với việc resume quá trình vừa xóa trước đó.

**Rollback** - Chúng ta có thể roll back lại deployment hoặc deployment trong process. người dùng có thể khởi tạo hoặc update lại deployment bằng cách sử dụng **DeploymentSpec.PodTemplateSpec = ordRC.PodTemplateSpec**.

### Chiến lược triển khai
Deployment strategies giúp cho việc xác định làm thế nào để RC mới thay thế được RC đang tồn tại.

**Rolling Update** - Update từng thành phần trong khi các thành phần khác vẫn chạy.

Để khởi tạo một deployment cho nginx webserver, ta chỉnh sửa lại file _nginx-deploy.yaml_:
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
  name: nginx
  namespace:
spec:
  minReadySeconds: 10
  progressDeadlineSeconds: 300
  revisionHistoryLimit: 3
  replicas: 6
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.12
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        readinessProbe:
          httpGet:
            path: /
            port: 80
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 10
          periodSeconds: 5
          successThreshold: 3
          failureThreshold: 1
      restartPolicy: Always
```
Trong đó:
  * _minReadySeconds_ xác định bao lâu một pod mới được khởi tạo sẽ sẵn sàng trước khi pod được sẵn sàng xử lí. Cho đến khi pod sẵn sàng trở lại, tiến trình cập nhật không thể tiếp tục. Một khi probe sẵn sàng và pod sẵn sàng, thì update có thể được tiếp dục.
và khởi tạo deployment.

  * _progressDeadlineSeconds_ xác định nếu sau một thời gian nào đó Pod không được xủ lí thì coi như bị chết.

  * _revisionHistoryLimit_ có bao nhiêu ReplicaSets cũ trong một Deployment mà bạn muốn giữ lại. Theo mặc định, nó là 10.

  * Trong phần _strategy_ ta cài đặt là rollingUpdate với 2 tham số:

    * _maxSurge_ : kiểm soát số lượng pod tồn tại trên số lượng bản sao mong muốn được cấu hình khi triển khai. Nó mặc định là 1, có nghĩa là có thể có nhiều nhất một thể hiện nhiều hơn số lượng mong muốn. Ví dụ: nếu số lượng bản sao mong muốn được đặt thành 3, sẽ không bao giờ có nhiều hơn 4 trường hợp pod chạy trong khi cập nhật cùng một lúc.

    * _maxUnavailable_ : kiểm soát số lượng pod có thể không có sẵn bên dưới với số lượng bản sao mong muốn trong quá trình cập nhật. Nó cũng mặc định là 1, có nghĩa là số lượng phiên bản nhóm khả dụng không bao giờ được giảm xuống dưới 1 so với số lượng bản sao mong muốn. Ví dụ: nếu số lượng bản sao mong muốn được đặt thành 3, sẽ luôn có ít nhất 2 trường hợp nhóm có sẵn để phục vụ các yêu cầu trong toàn bộ buổi giới thiệu.

```sh
kubectl create -f nginx-deploy.yaml 
deployment "nginx" created
```

Deployment khởi tạo đối tượng bằng lệnh sau:
```sh
kubectl get all -l run=nginx -o wide

NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES	SELECTOR
deploy/nginx   3         3         3            3           37s   nginx        nginx:1.12   run=nginx

NAME                  DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES .     SELECTOR
rs/nginx-698d6b8c9f   3         3         3       37s    nginx        nginx:1.12   pod-template

NAME                        READY     STATUS    RESTARTS   AGE       IP            NODE
po/nginx-698d6b8c9f-cj9n6   1/1       Running   0          37s       10.38.4.200   kubew04
po/nginx-698d6b8c9f-sr6fh   1/1       Running   0          37s       10.38.5.137   kubew05
po/nginx-698d6b8c9f-vpsm4   1/1       Running   0          37s       10.38.3.125   kubew03
```

Để cập nhật pods cùng với một phiên bản nginx image mới:
```sh
kubectl set image deploy nginx nginx=nginx:1.13
```

Kiểm tra trạng thái triển khai cập nhật:
```sh
kubectl rollout status deploy nginx
```

Nếu bạn muốn tạm dừng triển khai cập nhật:
```sh
kubectl rollout pause deploy nginx
```

Tiếp tục để hoàn thành:
```sh
kubectl rollout resume deploy nginx
```

Để downgrade, lùi lại rollout
```sh
kubectl rollout undo deploy nginx
```

Chú ý: Khi trình triển khai bị lỗi pod đầu tiên thì trình update sẽ bị treo. Điều có thể làm duy nhất lúc này là rollout lại.

