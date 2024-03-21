#### Хранение в K8s. Часть 1

###### 1).
У нас 2 контейнера, чтоб попасть в тот , который должен читать

```shell
kubectl exec -it test-786899794-xwtcl -c multitool -- sh
```
ДАлее
```
cat /data/log/output.txt
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
ps
    PID TTY          TIME CMD
     71 pts/0    00:00:00 sh
     84 pts/0    00:00:00 ps
```
Либо можно через log прочитать
```
kubectl logs test-786899794-xwtcl -c multitool                                                                                                                                            git:master*
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
Привет, мир $(date)
```
Чтоб попасть в основной контейнер, который строчить логи в данном случае

```
 kubectl exec -it test-786899794-xwtcl -c busybox -- sh
 ps
PID   USER     TIME  COMMAND
    1 root      0:00 sh -c mkdir -p /data/log && while true; do echo 'Привет, мир $(date)' >> /data/log/output.txt; sleep 8; done
   49 root      0:00 sh
   59 root      0:00 sleep 8
   60 root      0:00 ps
```
Сам манифест тут [manifest.yaml](manifest/manifest.yaml)

###### 2)

Сам [DaemonSet.yaml](manifest/DaemonSet.yaml)
```
kubectl exec -it fluentd-bct46 -- bash
root@fluentd-bct46:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
....
root@fluentd-bct46:/# cat /var/log/syslog | head -n 1
Mar 14 17:35:26 ubuntu-jammy kernel: [    0.000000] Linux version 5.15.0-71-generic (buildd@lcy02-amd64-044) (gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #78-Ubuntu SMP Tue Apr 18 09:00:29 UTC 2023 (Ubuntu 5.15.0-71.78-generic 5.15.92)
```

Время создания пода
```
kubectl get po
NAME            READY   STATUS    RESTARTS   AGE
fluentd-bct46   1/1     Running   0          2m26s
date
Чт 21 мар 2024 13:45:06 MSK
```