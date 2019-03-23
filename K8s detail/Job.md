## Jobs
Trong K8s, một Jobs được khởi tạo để đợi processes. Một Jobs khởi tạo 1 hay nhiều pods và đảm bảo rằng một trong số chúng đã thành công. Khi tất cả pod thành công, thì job hoàn thành.

Cho ví dụ, tạo file _hello-job.yaml_ xác định một bộ 16 pods và mỗi pod in ra một thông báo đơn giản. Trong trường hợp này, thực hiện song song 4 pods.
```sh
apiVersion: batch/v1
kind: Job
metadata:
  name: simplejob
spec:
  completions: 16
  parallelism: 4
  template:
    metadata:
      name: hello
    spec:
      containers:
      - name: hello
        image: busybox
        imagePullPolicy: IfNotPresent
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        args:
        - /bin/sh
        - -c
        - echo Hello from $(POD_NAME)
      restartPolicy: OnFailure
```

Kiểm tra job status:
```sh
kubectl get jobs -o wide
NAME           DESIRED   SUCCESSFUL   AGE       CONTAINERS   IMAGES    SELECTOR
simplejob      16         4           2m        hello        busybox 
```

Xóa một job là sẽ xóa tất cả các pods đã khởi tạo:
```sh
kubectl delete job simplejob
kubectl get pods -o wide -a
```