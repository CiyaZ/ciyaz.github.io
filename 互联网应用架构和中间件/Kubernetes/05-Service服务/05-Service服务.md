# Service服务

之前章节的例子中，我们知道Pod的IP地址是集群内部IP，外部不能访问，因此使用了Service的NodePort模式，实现了对外暴露Pod提供的应用程序。

k8s中，Service用于将一组Pod封装为一个服务，提供一个统一的入口，并在需要提供服务时按照特定的访问策略，访问后端Pod。Service工作于TCP/IP网络的传输层，支持TCP、UDP协议。

关于服务发现：我们都知道微服务的概念，在k8s中，Service负责封装服务，配合DNS插件，也可以实现服务发现功能，因此实际上k8s和SpringCloud在功能上具有一些重合的部分。

## Service的类型

* `ClusterIP`：通过一个集群内布IP提供服务
* `NodePort`：通过一个可供外部访问的IP提供服务，实际上是在每个Node上都打开了同一个端口
* `LoadBalance`：通过外部负载均衡器访问，一般是公有云的托管集群支持该模式

## Service例子

下面例子是我们之前使用过的一个Service配置文件，这里我们再详细介绍一下。

```yml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: sbdemo
  name: sbdemo
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: sbdemo
  type: NodePort
status:
  loadBalancer: {}
```

* `spec.selector`：标签选择器，该字段用于和Pod建立关联关系
* `spec.type`：Service的类型，可选值为前面介绍的三种
* `spec.ports[].protocol`：协议，支持TCP或UDP
* `spec.ports[].port`：集群内部提供服务的端口
* `spec.ports[].targetPort`：容器的端口
* `spec.ports[].nodePort`：在Node上打开的可供外部访问的端口，如未指定，则随机选择一个

## Service相关操作

通过yaml配置文件部署Service：
```
kubectl apply -f <yaml配置文件>
```

查看当前所有Service：
```
kubectl get svc
```

查看Service详细信息：
```
kubectl describe svc <Service名称>
```

删除Service：
```
kubectl delete svc <Service名称>
```
