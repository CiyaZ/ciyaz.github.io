# Deployment部署

k8s中，我们部署应用通常都不是直接创建Pod，而是通过Deployment来定义Pod，以及其副本数量，然后交给k8s自动调度。在之前部署例子中，我们已经演示了通过`--dry-run`来生成Deployment的配置文件模板。除此之外，通过操作Deployment还能实现升级、回退、扩容等操作。

## Deployment例子

下面例子中，部署了一个`1.20`版本的Nginx，并配置了3个副本。

deploy.yaml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    k8s-app: nginx-demo
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

上面配置中，我们可以看到，其实`template`中定义的就是Pod相关的内容。`spec.selector`需要能够匹配`spec.template.metadata.labels`，这样才能正常部署。

# Deployment基本操作

通过yaml配置文件部署Deployment：
```
kubectl apply -f <yaml配置文件>
```

`kubectl create`和`kubectl apply`的区别：

* `create`：删除现有的组件，根据配置文件重新生成
* `apply`：根据配置文件升级现有的组件

查看当前所有Deployment：
```
kubectl get deployment
```

查看Deployment详细信息：
```
kubectl describe deployment <Deployment名称>
```

删除Deployment：
```
kubectl delete deployment <Deployment名称>
```

注：删除Deployment后，对应Pod会自动销毁、删除。

## 重新部署

实际的线上环境因为情况非常复杂，我们可能遇到部署失败的情况，有时我们希望已有的Deployment重新尝试部署，可以使用如下命令实现：

```
kubectl rollout restart deployment <Deployment名称>
```

## 升级和回退版本

k8s能够按照一定的策略进行弹性升级，这里我们以上面部署的`nginx`为例进行操作。

查看升级操作状态：
```
kubectl rollout status deployment <Deployment名称>
```

输出结果例子：

```
Waiting for deployment "nginx-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deploy" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deploy" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deploy" successfully rolled out
```

查看升级操作历史：
```
kubectl rollout history deployment <Deployment名称>
```

输出结果例子：
```
deployment.apps/nginx-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

升级到指定镜象版本例子：
```
kubectl set image deployment nginx-deploy nginx=nginx:1.20.1
```

回退到指定版本例子：
```
kubectl rollout undo deployment nginx-deploy --to-revision=1
```

这里的`revision`版本号我们可以通过`kubectl rollout history`来找到。

## 伸缩副本数量

上面我们创建了3个副本，这里我们将其修改为5个副本：

```
kubectl scale deployment nginx-deploy --replicas=5
```

伸缩操作和软件镜像版本无关，因此不会计入`revision`中。
