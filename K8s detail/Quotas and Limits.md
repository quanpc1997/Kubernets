## Quotas and Limits
Namspaces giúp cho người dùng hoặc nhóm người dùng chia sẻ một cụm cùng với một số Node. Nó có thể là một mối quan hệ mà một team có thể sử dụng nhiều hơn phần tài nguyên. Resource quotas là công cụ để giải quyết mối quan hệ này.

Một resource quota quy định sự hạn chế để giới hạn việc sử dụng tài nguyên trên namespace. Nó có thể giới hạn số lượng các object mà có thể khởi tạo trong namespace. Cũng như tổng số tài nguyên tính toán có thể được sử dụng trong dự án đó. Các pod tham gia vào dự án đó sẽ phải tuân thủ tất cả các quy tắc. Nếu tổng các trị số quá một quy tắc nào đó thì pod đó không được tham gia vào namespaece. Mặt khác, khi giới hạn đã bị vượt quá thì không thể thêm bất cứ pod nào vào được.

Để định nghĩa một resource quota ta tạo file cấu hình _quota.yaml_ để quy định hạn chế trong namespace hiện tại.
```sh
apiVersion: v1
kind: ResourceQuota
metadata:
  name: project-quota
spec:
  hard:
    limits.memory: 1Gi
    limits.cpu: 1
    pods: 10
``` 

Tạo quota và kiểm tra namespace hiện tại:
```sh
kubectl config current-context
projectone/musa-cluster/admin

kubectl create -f quota.yaml
resourcequota "project-quota" created

kubectl describe ns projectone
Name:   projectone
Labels: type=project
Status: Active

Resource Quotas
 Name:          project-quota
 Resource       Used    Hard
 --------       ---     ---
 limits.cpu     0       1
 limits.memory  0       1Gi
 pods           0       10

No resource limits.
```

Ràng buộc trên giới hạn chỉ có 1 CPU, 1GB ram và tối đa 10 pods chạy.  
Cho ví dụ: 

Thử tạo một nginx pod
```sh
kubectl create -f nginx-pod.yaml
Error from server (Forbidden) ..
```
Giải thích cho điều này thì mặc định, một pod sẽ cố để chấp nhận tất cả CPU và RAM sẵn sàng trong hệ thống. Vì chúng ta đang đặt giới hạn tài nguyên trong namespace nên không thể yêu cầu không thể vượt quá giới hạn.

Để tạo pod trong trường hợp này ta cần giới hạn bằng cách sau. Tạo file cấu hình _nginx_deploy_limited.yaml_.
```sh
...
    spec:
      containers:
      - image: nginx:latest
        resources:
          limits:
            cpu: 200m
            memory: 512Mi
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
...
```
với việc đặt thông số như trên thì ta chỉ có thể tạo 2 pod. 

Để tránh việc trong một namespace có 1 pod nào đó chiếm quá nhiều tài nguyên. Ta có thể cấu hình như sau. Tạo file _limitrange.yaml_
```sh
---
kind: LimitRange
apiVersion: v1
metadata:
  name: container-limit-ranges
spec:
  limits:
  - type: Container
    max:
      cpu: 200m
      memory: 512Mi
    min:
      cpu:
      memory:
    default:
      cpu: 100m
      memory: 256Mi
```

Đoạn trên sẽ giới hạn lượng tài nguyên tối đa và tối thiểu mà pod có thể sử dụng.
```sh
kubectl create -f limitranges.yaml
limitrange "container-limit-ranges" created

kubectl describe namespace projectone
Name:   projectone
Labels: type=project
Status: Active

Resource Quotas
 Name:          project-quota
 Resource       Used    Hard
 --------       ---     ---
 limits.cpu     0       1
 limits.memory  0       1Gi
 pods           0       10

Resource Limits
 Type           Resource        Min     Max     Default Request Default Limit   Max Limit/Request Ratio
 ----           --------        ---     ---     --------------- -------------   -----------------------
 Container      memory          0       512Mi   256Mi           256Mi           -
 Container      cpu             0       200m    100m            100m            -
```

Thử tạo nginx pod:
```sh
...
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        cpu: 250m
        memory: 512Mi
    ports:
    - containerPort: 80
...
```

Kết quả lập tức báo lỗi.
```sh
kubectl create -f nginx-limited-pod.yaml
Error from server (Forbidden): error when creating "nginx-limited-pod.yaml":
pods "nginx" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m.
```

