# 集群部署

## kubectl操作

#### 依据配置文件进行创建

```shell
kubectl create -f [config_file]
```

如果后面的文件是`ns.yaml`，则创建命名空间，如果是`s.yaml`，则创建一个service，以此类推

#### 依据配置文件进行更新

```shell
kubectl replace -f [config_file]
```

同上，如：

```shell
kubectl replace -f d.yaml
```

#### 依据配置文件进行删除

```shell
kubectl delete -f [config_file]
```

同上

#### 获取一个命名空间下的pods

```shell
kubectl get pods -n [namespace_name]
```

如果使用`--all-namespaces`参数的话，则返回全部命名空间下的pods

```shell
kubectl get pods --all-namespaces
```

显示的结果：

```shell
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
elite         be-5b85994c5-qsffd                 1/1     Running   0          31d
elite         fe-54b8dd67f4-hg9vf                1/1     Running   0          124d
kstack        kstack-backend-7b79bb54bc-gl6bt    1/1     Running   0          53s
kube-system   coredns-857cdbd8b4-v747x           1/1     Running   0          233d
```

会返回每一个pod所属的命名空间，该pod唯一的name（用于唯一确定pod），是否就绪，当前状态如何，重启过几次，以及健康生存了多久。

#### 获取一个pod的日志

```shell
kubectl logs [pod_name] -n [namespace_name] [-f]
```

pod_name为上一步查看到的pod name，查看日志需要指定pod的命名空间，-f表示持续输出日志。如：

```shell
kubectl logs kstack-backend-7b79bb54bc-gl6bt -n kstack -f
```

