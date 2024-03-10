#### Запуск приложений в K8S
1).
Создать Deployment и обеспечить доступ к репликам приложения из другого Pod</br>
1.1) Оставлю манифесты в данном файле
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-netology
  labels:
    app: netology
spec:
  replicas: 1
  selector:
    matchLabels:
      app: netology
  template:
    metadata:
      labels:
        app: netology
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "1180"
        ports:
        - containerPort: 1180
          name: http-port
```
потребовалось указать env для корректной работы.
```shell
kubectl get deployment -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS        IMAGES                                 SELECTOR
nginx-netology   1/1     1            1           4m35s   nginx,multitool   nginx:latest,wbitt/network-multitool   app=netology

```

```shell
kubectl get po -o wide
NAME                              READY   STATUS    RESTARTS        AGE     IP            NODE     NOMINATED NODE   READINESS GATES
hello-world                       1/1     Running   5 (6m38s ago)   2d10h   10.1.219.86   master   <none>           <none>
netology-web                      1/1     Running   2 (6m38s ago)   2d6h    10.1.219.85   master   <none>           <none>
nginx-netology-685645b954-h2cht   2/2     Running   0               98s     10.1.219.88   master   <none>           <none>

```
1.2) Увеличить до 2
```shell
kubectl get deployment -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS        IMAGES                                 SELECTOR
nginx-netology   2/2     2            2           8m7s   nginx,multitool   nginx:latest,wbitt/network-multitool   app=netology
```
```shell
kubectl get po -o wide
NAME                              READY   STATUS    RESTARTS        AGE     IP            NODE     NOMINATED NODE   READINESS GATES
hello-world                       1/1     Running   5 (9m40s ago)   2d10h   10.1.219.86   master   <none>           <none>
netology-web                      1/1     Running   2 (9m40s ago)   2d6h    10.1.219.85   master   <none>           <none>
nginx-netology-685645b954-h2cht   2/2     Running   0               4m40s   10.1.219.88   master   <none>           <none>
nginx-netology-685645b954-zpxmd   2/2     Running   0               59s     10.1.219.89   master   <none>           <none>
```
Итоговый манифест [тут](manifest/deployment_1.1.yaml)</br>
1.4) Создать Service</br>
[service](manifest/service_1.1.yaml)
```shell
kubectl get svc -o wide
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE    SELECTOR
kubernetes     ClusterIP   10.152.183.1    <none>        443/TCP    4d2h   <none>
netology-svc   ClusterIP   10.152.183.98   <none>        9090/TCP   18s    app=netology
```
1.5)
Сделал через запуск команды в поде
```shell
kubectl exec -it test-curl -- /bin/sh
 # curl netology-svc:9090
<!DOCTYPE html>
<html>
<head>
```
[Манифест пода](manifest/pod_1.1.yaml)</br>
2.)
```
nginx-netology-2-8574c88496-hbkb4   0/1     Init:0/1   0             50s
```

Манифест [Deployment](manifest/deployment_2.yaml)
Манифест [Service](manifest/service_2.yaml)

Но у меня проблема, так как тестирую в vagrant локально, то тут 2 сетевых интерфейса и из-за default интерфейса выползают сетевые ошибки, к примеру:

```shell
 kubectl logs nginx-netology-2-8dc6f4d66-d7jfm
Defaulted container "nginx" out of: nginx, init (init)
Error from server: Get "https://10.0.2.15:10250/containerLogs/default/nginx-netology-2-8dc6f4d66-d7jfm/nginx": dial tcp 10.0.2.15:10250: connect: network is unreachable
```
И далее не проходит, поды после удаления находятся в статусе Terminating и их никак не удалить.

Да busybox поставил версия 1.28 и пошло

```
mshapovalov@vm-1:~$ kubectl get po
NAME                                READY   STATUS     RESTARTS   AGE
nginx-netology-2-5dd955c8d7-zgd7g   0/1     Init:0/1   0          14s
mshapovalov@vm-1:~$ kubectl get po
NAME                                READY   STATUS            RESTARTS   AGE
nginx-netology-2-5dd955c8d7-zgd7g   0/1     PodInitializing   0          21s
mshapovalov@vm-1:~$ kubectl get po
mshapovalov@vm-1:~$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-netology-2-5dd955c8d7-zgd7g   1/1     Running   0          29s
```

```
kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP    20m
test-2svc    ClusterIP   10.152.183.68   <none>        8080/TCP   6m33s
```