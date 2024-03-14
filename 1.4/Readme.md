### Сетевое взаимодействие в K8S. Часть 1

###### 1
```shell
1.4 ➤ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP             6m47s
network      ClusterIP   10.152.183.22   <none>        9001/TCP,9002/TCP   73s
1.4 ➤ kubectl get po
NAME                             READY   STATUS    RESTARTS   AGE
nginx-netology-fb976dbb7-cwk6p   2/2     Running   0          6s
nginx-netology-fb976dbb7-jj6h2   2/2     Running   0          2m39s
nginx-netology-fb976dbb7-nh5pz   2/2     Running   0          2m39s
```
[Манифест](manifest/deployment-1.4.1.yaml)

```shell
kubectl run -it --rm --image=curlimages/curl curly -- sh                                                                                                          git:master*
If you don't see a command prompt, try pressing enter.
~ $ curl network:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
curl network:9002
WBITT Network MultiTool (with NGINX) - nginx-netology-fb976dbb7-nh5pz - 10.1.77.4 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
###### 2

```shell
kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP             40m
myapp        NodePort    10.152.183.62   <none>        80:30007/TCP        10s
network      ClusterIP   10.152.183.22   <none>        9001/TCP,9002/TCP   34m
1.4 ➤ curl 192.168.8.10:30007
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
```
Сам сервисе доступен [тут](manifest/service-nodeport.yaml)