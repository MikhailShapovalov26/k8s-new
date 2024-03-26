#### Конфигурация приложений
###### 1) Deployment приложения
[Манифест сам доступен тут](kubemanifesrs/Deployment_1.1.yaml)

``check``

Внутри контейнера с multitool
```
kubectl exec -it myapp-8689d5444f-8kp48 -c multitool -- sh                                                                                  
/ # echo $HTTP_PORT
8080
/ # curl localhost:8080
WBITT Network MultiTool (with NGINX) - myapp-8689d5444f-8kp48 - 10.1.77.27 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

Внутри контейнера с nginx
```
root@myapp-7bf7595cf9-crct5:/# cat /usr/share/nginx/html/index.html
"Hello Netology"
root@myapp-7bf7595cf9-crct5:/# curl localhost
"Hello Netology"
```
Сервис для проверки
```
kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    2d
myapp        ClusterIP   10.152.183.140   <none>        9090/TCP   26s

curl 10.152.183.140:9090
"Hello Netology"
```
###### 2) Создать приложение с вашей веб-страницей, доступной по HTTPS

[Сам манифест для изучения тут](kubemanifesrs/Deployment_1.2.yaml) 

Сертификата сгенерил данной командой
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=foo.bar.com"

```

Данной командой я отправил сертификаты в K8S, так как подставлять их в yaml было сложным заданием.
```
kubectl create secret tls ssl-tls  --cert=tls.crt --key=tls.key
```
Далее проверяем по итам, что у нас прошло
* Заметим что я передал Запись А в HOSTS, ее необходимо прописать в /etc/hosts

```
kubectl get ingress -A
NAMESPACE   NAME          CLASS    HOSTS         ADDRESS     PORTS     AGE
default     tls-ingress   public   foo.bar.com   127.0.0.1   80, 443   8m59s
----
kubectl describe  ingress tls-ingress
Name:             tls-ingress
Labels:           <none>
Namespace:        default
Address:          127.0.0.1
Ingress Class:    public
Default backend:  <default>
TLS:
  ssl-tls terminates foo.bar.com
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com  
               /   label:9091 (10.1.77.38:80)
Annotations:   <none>
Events:        <none>
```
Посмотреть список секретов(сертификатов)

```
kubectl describe secret ssl-tls
Name:         ssl-tls
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1119 bytes
tls.key:  1704 bytes

```
Результат тестирования на локальной машине

```
curl -k https://foo.bar.com
"Hello Netology"
```