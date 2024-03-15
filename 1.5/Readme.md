#### Сетевое взаимодействие в K8S. Часть 2

##### 1
```
kubectl exec -it pod/backend-86449998c6-kqzjd -- /bin/bash
backend-86449998c6-kqzjd:/# curl service-frontend:8081
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
....
```
```
kubectl exec -it pod/frontend-57f6d7f58f-n5cdn -- /bin/bash
root@frontend-57f6d7f58f-n5cdn:/# curl service-backend:8082
WBITT Network MultiTool (with NGINX) - backend-86449998c6-kqzjd - 10.1.77.30 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
Манифесты [deployment-backend](manifests/deployment-backend.yaml) [deployment-frontenf](manifests/deployment-frontenf.yaml) [services-backend](manifests/services-backend.yaml) [services-frontend](manifests/services-frontend.yaml)

#### 2  Ingress

```
microk8s enable ingress
```

На самом сервере выполнял данные команды
```
curl -k  http://127.0.0.1:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
...
```
```
curl -k  http://127.0.0.1:80/api
WBITT Network MultiTool (with NGINX) - backend-86449998c6-kqzjd - 10.1.77.30 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
Сам манифест [Ingress тут](manifests/ingress.yaml)