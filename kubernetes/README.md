# k8s
## minikube

```shell
# 不适用虚拟机，裸机运行
minikube config set vm-driver none
minikube config set driver none

# 启动，加上阿里云镜像
sudo minikube start --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

## kubectl

```shell
# 列出pod
kubectl get pods
kubectl get pods -n {$namespace}
kubectl get pods --all-namespaces
# 查看详情信息
kubectl get pods -o wide
# 查看命名空间
kubectl get ns
# 查看service
kubectl get svc
kubectl get svc -n {$namespace}
# 查看ingress
kubectl get ing

kubectl create -f {$file}

kubectl logs {$podName} -n {$namespace}
# 实时日志
kubectl logs {$podName} -n {$namespace} -f


# 基于 pod.yaml 定义的名称删除 pod 
kubectl delete -f pod.yaml 

# 删除所有 Pod
kubectl delete pod --all
```