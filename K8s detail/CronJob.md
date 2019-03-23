## Cron Jobs
Một Crom Job là một công việc được quản lý dựa trên thời gian. Một Cron job chạy định kì theo một lịch trình nhất định được viết bằng crom unix format tiêu chuẩn.

Cho ví dụ, _date-cronjob.yaml_ file được xác định là một cron job để tin cần in ngày giờ hiện tại trên apiVersion: batch/v1beta1

```sh
kind: CronJob
metadata:
  name: currentdate
spec:
  schedule: "*/1 * * * *"           //Thời gian kiểu crontab
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: date
            image: busybox
            imagePullPolicy: IfNotPresent
            args:
            - /bin/sh
            - -c
            - echo "Current date is"; date
          restartPolicy: OnFailure
```

Khỏi tạo một cron job
```sh
kubectl create -f data-cronjob.yaml 
```
và kiểm tra pod được khởi tạo.
```sh
kubectl get pods -o wide -a 
NAME                           READY     STATUS      RESTARTS   AGE       IP            NODE
currentdate-1508917200-j8vl9   0/1       Completed   0          2m        10.38.3.127   node1
currentdate-1508917260-qg9zn   1/1       Running     0          1m        10.38.5.98    node2
```

