### Helm

##### Задание 1. Подготовить Helm-чарт для приложения

Сам templates [можно посмотреть тут](netology/templates/deployment.yaml) 
а [values тут](netology/values.yaml)
```
helm install netology ./netology -n default                                                                                           
NAME: netology
LAST DEPLOYED: Tue Apr  2 10:28:44 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
```
kubectl get po 
NAME                             READY   STATUS    RESTARTS      AGE
app-deployment-885fb8c8d-zhbgn   1/1     Running   0             76s
```
describe
```
  Normal  Scheduled  2m30s  default-scheduler  Successfully assigned default/app-deployment-885fb8c8d-zhbgn to k8s
  Normal  Pulling    2m30s  kubelet            Pulling image "nginx:1.16"
  Normal  Pulled     2m20s  kubelet            Successfully pulled image "nginx:1.16" in 9.508s (9.508s including waiting)
  Normal  Created    2m20s  kubelet            Created container app-container
  Normal  Started    2m20s  kubelet            Started container app-container
```
Версия указана, та что указал в values image tag

##### Задание 2. Запустить две версии в разных неймспейсах
```
helm upgrade --install netology ./netology -n app2 --set deploymentName='backend'

Release "netology" has been upgraded. Happy Helming!
NAME: netology
LAST DEPLOYED: Tue Apr  2 10:48:04 2024
NAMESPACE: app2
STATUS: deployed
REVISION: 2
```
```
helm install new-release-name  netology -n app2 --set deploymentName='front'

NAME: new-release-name
LAST DEPLOYED: Tue Apr  2 10:49:54 2024
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
```
kubectl get po -n app2
NAME                                  READY   STATUS    RESTARTS   AGE
backend-deployment-684c6fdf48-8l5nm   1/1     Running   0          2m15s
front-deployment-75df8ccb48-mbqd6     1/1     Running   0          25s
```

```
helm install version-3  netology -n app3 --set deploymentName='test'

NAME: version-3
LAST DEPLOYED: Tue Apr  2 11:29:27 2024
NAMESPACE: app3
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
```
kubectl get po -n app3
NAME                              READY   STATUS    RESTARTS   AGE
test-deployment-bdd967d78-vvzrb   1/1     Running   0          28s
```