#### Обновление приложений


Статьи на память.
1) https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment
2) https://habr.com/ru/companies/flant/articles/471620/

Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор
Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
Запас по ресурсам в менее загруженный момент времени составляет 20%.
Обновление мажорное, новые версии приложения не умеют работать со старыми.
Вам нужно объяснить свой выбор стратегии обновления приложения.

Ответ:

В данном случае имеет важное значение только пункт 4, а именно `Обновление мажорное, новые версии приложения не умеют работать со старыми.`</br>
Тут подойдет только `Recreate (повторное создание)`

Соответствующий манифест выглядит примерно так:
```
spec:
  replicas: 3
  strategy:
    type: Recreate
  template:
```
- старые pod'ы убиваются все разом и заменяются новыми

Задание 2. Обновить приложение
Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.
Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
Откатиться после неудачного обновления.

в данном случае будем использовать стандартный вариант 
`Rolling (постепенный, «накатываемый» деплой)`

вариант который я добавил в свой манифест 
```
spec:
...
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxSurge: 25%
       maxUnavailable: 25%  
  template:
```
Это стандартная стратегия развертывания в Kubernetes. Она постепенно, один за другим, заменяет pod'ы со старой версией приложения на pod'ы с новой версией — без простоя кластера.
```
kubectl get po
NAME                      READY   STATUS    RESTARTS   AGE
backend-87ff4b98f-52d4f   2/2     Running   0          3m39s
backend-87ff4b98f-nrsgj   2/2     Running   0          3m39s
backend-87ff4b98f-tk2f6   2/2     Running   0          3m39s
backend-87ff4b98f-vwt7g   2/2     Running   0          3m39s
backend-87ff4b98f-zj6ss   2/2     Running   0          3m39s
debian@k8s-master1:~$ kubectl rollout status deployment/backend
deployment "backend" successfully rolled out
debian@k8s-master1:~$ kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
backend   5/5     5            5           4m57s
```

```
kubectl get rs
NAME                DESIRED   CURRENT   READY   AGE
backend-87ff4b98f   5         5         5       5m23s

```

```
kubectl rollout history deployment/backend
deployment.apps/backend 
REVISION  CHANGE-CAUSE
1         <none>

```

После обновления nginx на версию выше видим сл вывод
```
kubectl get po -w
NAME                       READY   STATUS              RESTARTS   AGE
backend-6ff5d5f75c-9lnbl   0/2     ContainerCreating   0          2s
backend-6ff5d5f75c-pbzkd   0/2     ContainerCreating   0          2s
backend-6ff5d5f75c-tvdpb   0/2     ContainerCreating   0          2s
backend-87ff4b98f-22z97    2/2     Running             0          3m12s
backend-87ff4b98f-fjqmz    2/2     Running             0          3m12s
backend-87ff4b98f-srx6m    2/2     Running             0          3m12s
backend-87ff4b98f-z9pxv    2/2     Running             0          3m12s
backend-6ff5d5f75c-9lnbl   2/2     Running             0          13s
backend-87ff4b98f-z9pxv    2/2     Terminating         0          3m23s
backend-6ff5d5f75c-5lnd7   0/2     Pending             0          0s
backend-6ff5d5f75c-5lnd7   0/2     Pending             0          0s
backend-6ff5d5f75c-5lnd7   0/2     ContainerCreating   0          0s
backend-6ff5d5f75c-tvdpb   2/2     Running             0          14s
backend-87ff4b98f-22z97    2/2     Terminating         0          3m24s
backend-6ff5d5f75c-6hf75   0/2     Pending             0          0s
backend-6ff5d5f75c-6hf75   0/2     Pending             0          0s
backend-6ff5d5f75c-6hf75   0/2     ContainerCreating   0          0s
backend-6ff5d5f75c-5lnd7   0/2     ContainerCreating   0          4s
backend-87ff4b98f-22z97    2/2     Terminating         0          3m27s
backend-87ff4b98f-z9pxv    2/2     Terminating         0          3m27s
backend-87ff4b98f-22z97    0/2     Terminating         0          3m27s
backend-87ff4b98f-z9pxv    0/2     Terminating         0          3m27s
backend-6ff5d5f75c-6hf75   0/2     ContainerCreating   0          3s
backend-87ff4b98f-z9pxv    0/2     Terminating         0          3m28s
backend-87ff4b98f-z9pxv    0/2     Terminating         0          3m28s
....
```
Обновление прошло успешно
```
kubectl get po
NAME                       READY   STATUS    RESTARTS   AGE
backend-6ff5d5f75c-5lnd7   2/2     Running   0          42s
backend-6ff5d5f75c-6hf75   2/2     Running   0          41s
backend-6ff5d5f75c-9lnbl   2/2     Running   0          55s
backend-6ff5d5f75c-pbzkd   2/2     Running   0          55s
backend-6ff5d5f75c-tvdpb   2/2     Running   0          55s
```
К примеру в 1 из подов
```
Successfully pulled image "nginx:1.20" in 7.836s (9.913s including waiting)
```
После делаем версию nginx 1.280
```
kubectl get po -w
NAME                       READY   STATUS              RESTARTS   AGE
backend-6ff5d5f75c-5lnd7   2/2     Running             0          5m8s
backend-6ff5d5f75c-6hf75   2/2     Running             0          5m7s
backend-6ff5d5f75c-9lnbl   2/2     Running             0          5m21s
backend-6ff5d5f75c-tvdpb   2/2     Running             0          5m21s
backend-74d884cff9-67vw9   0/2     ContainerCreating   0          3s
backend-74d884cff9-klc78   0/2     ContainerCreating   0          3s
backend-74d884cff9-nbgx7   0/2     ContainerCreating   0          3s
backend-74d884cff9-nbgx7   1/2     ErrImagePull        0          6s
backend-74d884cff9-67vw9   1/2     ErrImagePull        0          7s

```
По итогу видим что 4 пода подняты и работают 
```
kubectl get po
NAME                       READY   STATUS         RESTARTS   AGE
backend-6ff5d5f75c-5lnd7   2/2     Running        0          6m15s
backend-6ff5d5f75c-6hf75   2/2     Running        0          6m14s
backend-6ff5d5f75c-9lnbl   2/2     Running        0          6m28s
backend-6ff5d5f75c-tvdpb   2/2     Running        0          6m28s
backend-74d884cff9-67vw9   1/2     ErrImagePull   0          70s
backend-74d884cff9-klc78   1/2     ErrImagePull   0          70s
backend-74d884cff9-nbgx7   1/2     ErrImagePull   0          70s
```
```
kubectl rollout status deployment backend
Waiting for deployment "backend" rollout to finish: 3 out of 5 new replicas have been updated...
```
```
kubectl get rs
NAME                 DESIRED   CURRENT   READY   AGE
backend-6ff5d5f75c   4         4         4       8m33s
backend-74d884cff9   3         3         0       3m15s
backend-87ff4b98f    0         0         0       11m

```
history
```
kubectl rollout history deployment backend
deployment.apps/backend 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

Откат к предыдущему варианту
```
kubectl rollout undo deployment backend --to-revision=2
deployment.apps/backend rolled back
debian@k8s-master1:~$ kubectl rollout history deployment backend
deployment.apps/backend 
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>
kubectl get po
NAME                       READY   STATUS    RESTARTS   AGE
backend-6ff5d5f75c-5lnd7   2/2     Running   0          11m
backend-6ff5d5f75c-6hf75   2/2     Running   0          11m
backend-6ff5d5f75c-9lnbl   2/2     Running   0          12m
backend-6ff5d5f75c-9lqmw   2/2     Running   0          33s
backend-6ff5d5f75c-tvdpb   2/2     Running   0          12m
kubectl describe po backend-6ff5d5f75c-9lnbl | grep nginx
  nginx:
    Image:          nginx:1.20
    Image ID:       docker.io/library/nginx@sha256:38f8c1d9613f3f42e7969c3b1dd5c3277e635d4576713e6453c6193e66270a6d
  Normal  Pulling    12m   kubelet            Pulling image "nginx:1.20"
  Normal  Pulled     12m   kubelet            Successfully pulled image "nginx:1.20" in 7.836s (9.913s including waiting)

```

Задание 3*. Создать Canary deployment
Создать два deployment'а приложения nginx.
При помощи разных ConfigMap сделать две версии приложения — веб-страницы.
С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.