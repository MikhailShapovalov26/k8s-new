### Установка Кластера K8S
Установку произвёл с помощью [ansible роли](K8S) которую написал самостоятельно.
На ОС Дебиан
```
debian@k8s-master:~$ kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   23m   v1.29.3
k8s-worker1   Ready    <none>          58s   v1.29.3
k8s-worker2   Ready    <none>          61s   v1.29.3
debian@k8s-master:~$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
Дополнительно
```
kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION    CONTAINER-RUNTIME
k8s-master    Ready    control-plane   30m     v1.29.3   10.1.11.10    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.28
k8s-worker1   Ready    <none>          8m28s   v1.29.3   10.1.11.11    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.28
k8s-worker2   Ready    <none>          8m31s   v1.29.3   10.1.11.12    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.28
```

Тестовый под для проверки
```
kubectl get po -o wide
NAME        READY   STATUS              RESTARTS   AGE   IP       NODE          NOMINATED NODE   READINESS GATES
hazelcast   0/1     ContainerCreating   0          11s   <none>   k8s-worker1   <none>           <none>

kubectl get po -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
hazelcast   1/1     Running   0          43s   172.16.194.65   k8s-worker1   <none>           <none>
```

### 2
Установим несколько мастер нод, для этого пришлось усовершенствовать [ансибл роль](K8S/k8s.yaml).
```
kubectl get nodes -o wide
NAME          STATUS     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION    CONTAINER-RUNTIME
k8s-master1   Ready      control-plane   20m   v1.29.3   10.1.11.10    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
k8s-master2   NotReady   control-plane   40s   v1.29.3   10.1.11.11    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
k8s-master3   Ready      control-plane   55s   v1.29.3   10.1.11.12    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
```
```
kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION    CONTAINER-RUNTIME
k8s-master1   Ready    control-plane   23m     v1.29.3   10.1.11.10    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
k8s-master2   Ready    control-plane   3m15s   v1.29.3   10.1.11.11    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
k8s-master3   Ready    control-plane   3m30s   v1.29.3   10.1.11.12    <none>        Debian GNU/Linux 11 (bullseye)   5.10.0-10-amd64   containerd://1.6.31
```
