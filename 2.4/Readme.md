### Управление доступом

Необходимо дополнительно поставить !

```
microk8s enable rbac
```

Создать и подписать ssl сертификат

```
openssl genrsa -out IINetology.key 2048

openssl req -new -key IINetology.key -out IINetology.csr -subj "/CN=IINetology/O=Admins"

openssl x509 -req -in IINetology.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out IINetology.crt -days 500
```
*Делать всё надо на node-control master node.

1) Создайте пользователя внутри Kubernetes:
```
kubectl config set-credentials IINetology \
--client-certificate=IINetology.crt \
--client-key=IINetology.key

User "IINetology" set.
```
2) Задайте контекст для пользователя:
```
vagrant@k8s:~$ kubectl config set-context IINetology-context \
--cluster=microk8s-cluster --user=IINetology
Context "IINetology-context" created
```
*Стоит обратить внимания на --cluster

3) Переключаемся на наш контекст
```
vagrant@k8s:~$ kubectl config use-context IINetology-context
Switched to context "IINetology-context".
vagrant@k8s:~$ kubectl get po
Error from server (Forbidden): pods is forbidden: User "IINetology" cannot list resource "pods" in API group "" in the namespace "default"
vagrant@k8s:~$ kubectl get namespace
Error from server (Forbidden): namespaces is forbidden: User "IINetology" cannot list resource "namespaces" in API group "" at the cluster scope
```

После чего [прогоняем манифест](role.yaml)

list pods namespace default
```
vagrant@k8s:~$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
hello-world   1/1     Running   0          2m32s
```
describe pod
```
kubectl describe po
Name:             hello-world
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s/10.0.2.15
Start Time:       Fri, 29 Mar 2024 19:07:57 +0000
Labels:           app=hello-world
Annotations:      cni.projectcalico.org/containerID: 6845c839846a708d873386d873a78220740cb8acb1c3e9f34cab173fe3ebfaa8
                  cni.projectcalico.org/podIP: 10.1.77.46/32
                  cni.projectcalico.org/podIPs: 10.1.77.46/32
Status:           Running
IP:               10.1.77.46
IPs:
  IP:  10.1.77.46
```
log pod
```
kubectl logs hello-world
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
......
```
svc
```
kubectl get svc
Error from server (Forbidden): services is forbidden: User "IINetology" cannot list resource "services" in API group "" in the namespace "default"
```
delete pod
```
kubectl delete pod/hello-world
Error from server (Forbidden): pods "hello-world" is forbidden: User "IINetology" cannot delete resource "pods" in API group "" in the namespace "default"
```

Источники которые полезно сохранить
https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b
https://habr.com/ru/companies/flant/articles/470503/