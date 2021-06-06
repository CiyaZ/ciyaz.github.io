# Job单次任务和定时任务

之前我们部署的程序，如Nginx、我们自己写的SpringBoot程序等，都是持续运行的服务，除此之外k8s也支持单次任务（Job）和定时任务（CronJob）。

## Job单次任务

Job也会创建一个Pod，通过该Pod的返回码来判断其是否执行成功，如果发生异常退出，会将其重新启动。

### Job例子

下面是一个Job的例子：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  template:
    metadata:
      labels:
        app: myjob
    spec:
      restartPolicy: OnFailure
      containers:
      - name: busybox
        image: busybox
        args:
        - /bin/sh
        - -c
        - echo helloWorld
```

这里我们使用`busybox`镜象输出一段信息作为我们的Job。

注意：Job属于batch API组，因此这里`apiVersion`为`batch/v1`。

### Job相关操作

部署Job，我们直接使用`kubectl create`命令：

```
kubectl create -f <yaml配置文件名>
```

查看Job运行状态：

```
kubectl get job
```

其中，`STATUS`为`Complated`为执行成功。

## CronJob定时任务

CronJob通过指定一个Linux的Crontab表达式，来定时执行任务。例子：

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: myjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo helloWorld
```

CronJob每一次运行，就会创建一个新的Pod，并执行我们指定的内容。

我们可以用如下命令查看定时任务的执行情况：

```
kubectl get cronjob
```

如果不希望CronJob继续运行，可以将其删除：

```
kubectl delete cronjob <CronJob名>
```
