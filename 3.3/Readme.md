#### Как работает сеть в K8s

```
kubectl get po -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-658d97c59c-pcvpb   1/1     Running   0          7m46s
kube-system   calico-node-658br                          1/1     Running   0          7m46s
kube-system   coredns-76f75df574-645c7                   1/1     Running   0          7m51s
kube-system   coredns-76f75df574-bfg7w                   1/1     Running   0          7m51s
kube-system   etcd-k8s-master1                           1/1     Running   0          8m4s
kube-system   kube-apiserver-k8s-master1                 1/1     Running   0          8m4s
kube-system   kube-controller-manager-k8s-master1        1/1     Running   0          8m4s
kube-system   kube-proxy-8kvxs                           1/1     Running   0          7m51s
kube-system   kube-scheduler-k8s-master1                 1/1     Running   0          8m4s

```
Такие поды [поднял с помощью манифеста](manifests/Deployment.yaml)
```
kubectl get po -n app -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
backend-7bfbc88cc4-8xqwh   1/1     Running   0          96s   172.16.194.65   k8s-worker1   <none>           <none>
cache-658d4c944-vdr68      1/1     Running   0          96s   172.16.194.67   k8s-worker1   <none>           <none>
frontend-bc68bcf86-2wfzm   1/1     Running   0          96s   172.16.194.66   k8s-worker1   <none>           <none>
```
Сначала [запрещяем всё](manifests/deny-all-traffic.yaml) и далее начинаем навешывать разрешения [frontend->backend](manifests/networkpolicy_f-b.yaml)
Проверим
попадаем к примеру на под с фронтендом и проверяем бэкенд и каш, и видим что с фронтенда доступен только под с бэком
```
kubectl exec -it pod/frontend-bc68bcf86-2wfzm -n app -- bash

frontend-bc68bcf86-2wfzm:/# curl 172.16.194.65:8080
WBITT Network MultiTool (with NGINX) - backend-7bfbc88cc4-8xqwh - 172.16.194.65 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
frontend-bc68bcf86-2wfzm:/# curl 172.16.194.67:8080
^C
```

Ну и связка [backend->cache](manifests/networkpolicy_b-c.yaml)
```
kubectl exec -it pod/backend-7bfbc88cc4-8xqwh -n app -- bash

backend-7bfbc88cc4-8xqwh:/# curl 172.16.194.67:8080
WBITT Network MultiTool (with NGINX) - cache-658d4c944-vdr68 - 172.16.194.67 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
backend-7bfbc88cc4-8xqwh:/# curl 172.16.194.66:8080
^C
backend-7bfbc88cc4-8xqwh:/# 

```
По итогу на сервере появились такие правила
```
kubectl get networkpolicy -n app
NAME                        POD-SELECTOR   AGE
allow-backend-to-cache      app=cache      40s
allow-frontend-to-backend   app=backend    13m
deny-all-traffic            <none>         17m

```