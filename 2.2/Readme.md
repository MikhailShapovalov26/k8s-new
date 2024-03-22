#### Хранение в K8s. Часть 2

###### 1).


```
new_netology ➤ kubectl get pv                                                                                                                                                                   git:master*
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-test   1Gi        RWO            Retain           Bound    default/pvc-test                  <unset>                          72s
new_netology ➤ kubectl get pvc                                                                                                                                                                  git:master*
NAME       STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-test   Bound    pv-test   1Gi        RWO                           <unset>                 74s
new_netology ➤ kubectl get po                                                                                                                                                                   git:master*
NAME                    READY   STATUS    RESTARTS   AGE
test-7b48f685f8-bkbhw   2/2     Running   0          22s
```

```
less /mnt/data/log/output.txt  | tail -n 1
Привет, мир Fri Mar 22 14:30:11 UTC 2024
```

```
kubectl delete -f 2.2/manifest/manifest\ copy.yaml                                                                                                                               git:master*
persistentvolumeclaim "pvc-test" deleted
deployment.apps "test" deleted
new_netology ➤ kubectl get pv                                                                                                                                                                   git:master*
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM              STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-test   1Gi        RWO            Retain           Released   default/pvc-test                  <unset>                          44m
new_netology ➤ kubectl get pvc                                                                                                                                                                  git:master*
No resources found in default namespace.
new_netology ➤ kubectl get po                                                                                                                                                                   git:master*
No resources found in default namespace.
```

Полсе удаления Deployment и PVC

```
less /mnt/data/log/output.txt  | tail -n 1
Привет, мир Fri Mar 22 14:40:51 UTC 2024
```
Файл все ёщё на виртуальной машине
Данные всё ещё остались на виртуальной машине, так как я указал

> Retain</br>
The Retain reclaim policy allows for manual reclamation of the resource. When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered “released”. But it is not yet available for another claim because the previous claimant’s data remains on the volume. An administrator can manually reclaim the volume with the following steps.

> Delete the PersistentVolume. The associated storage asset in external infrastructure (such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume) still exists after the PV is deleted.
Manually clean up the data on the associated storage asset accordingly.
Manually delete the associated storage asset, or if you want to reuse the same storage asset, create a new PersistentVolume with the storage asset definition.

[Источник](https://k8s-docs.netlify.app/en/docs/concepts/storage/persistent-volumes/)

[Манифест на задания](manifest/manifest.yaml)

###### 2)

Ссылка на установку https://microk8s.io/docs/how-to-nfs

Комментарии по настройке
```
cat /etc/exports
/srv/nfs *(rw,sync,no_subtree_check)
```
Поставить * всесто IP адреса.

В самом StorageClass
```yaml
server: 192.168.8.10
```
Сервер где nfs server

Тестирования из пода на запись файлов
```
 cat /data2/log/output.txt 
Привет, мир Sun Mar 24 07:57:02 UTC 2024
Привет, мир Sun Mar 24 07:57:24 UTC 2024
Привет, мир Sun Mar 24 07:57:32 UTC 2024
```

Чтения данных с ноды 
```
cat /srv/nfs/pvc-e631a977-90ac-4ff5-8a9b-8a55a25ac361/log/output.txt 
Привет, мир Sun Mar 24 07:57:02 UTC 2024
Привет, мир Sun Mar 24 07:57:24 UTC 2024
Привет, мир Sun Mar 24 07:57:32 UTC 2024
Привет, мир Sun Mar 24 07:58:02 UTC 2024

```
Сам манифест с настройками [тут](manifest/manifest_2.yaml)