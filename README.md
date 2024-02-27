# Домашнее задание к занятию «Хранение в K8s. Часть 2»
- Цель задания: В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов
## Задание 1
- Что нужно сделать
- Создать Deployment приложения, использующего локальный PV, созданный вручную. [local-pv](), [local-pvc]()
- Создать Deployment приложения, состоящего из контейнеров busybox и multitool. [Deployment]()
- Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
- Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории.
- Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
- Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV. Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
- Предоставить манифесты, а также скриншоты или вывод необходимых команд.
```
vagrant@vagrant:~/kube/zad7$ kubectl apply -f local-pv.yaml
persistentvolume/local-pv created
vagrant@vagrant:~/kube/zad7$ sudo nano local-pvc.yaml
vagrant@vagrant:~/kube/zad7$ kubectl apply -f local-pvc.yaml
persistentvolumeclaim/local-pvc created
vagrant@vagrant:~/kube/zad7$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
local-pv   1Gi        RWO            Retain           Bound    default/local-pvc                           6m51s
vagrant@vagrant:~/kube/zad7$ kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
local-pvc   Bound    local-pv   1Gi        RWO                           6m
vagrant@vagrant:~/kube/zad7$ kubectl get pods
NAME                              READY   STATUS    RESTARTS        AGE
multitool-pod                     1/1     Running   70 (21m ago)    3d1h
app-deployment-5685d77995-7tzd2   2/2     Running   0               29s
```
```
vagrant@vagrant:~/kube/zad7$ kubectl exec -it app-deployment-5685d77995-7tzd2 -- cat /data/data.txt
Defaulted container "busybox" out of: busybox, multitool
Tue Feb 27 15:23:51 UTC 2024
Tue Feb 27 15:23:56 UTC 2024
Tue Feb 27 15:24:01 UTC 2024
Tue Feb 27 15:24:06 UTC 2024
Tue Feb 27 15:24:11 UTC 2024
Tue Feb 27 15:24:16 UTC 2024
Tue Feb 27 15:24:21 UTC 2024
Tue Feb 27 15:24:26 UTC 2024
Tue Feb 27 15:24:31 UTC 2024
Tue Feb 27 15:24:36 UTC 2024
Tue Feb 27 15:24:41 UTC 2024
Tue Feb 27 15:24:46 UTC 2024
Tue Feb 27 15:24:51 UTC 2024
Tue Feb 27 15:24:57 UTC 2024
Tue Feb 27 15:25:02 UTC 2024
Tue Feb 27 15:25:07 UTC 2024
Tue Feb 27 15:25:12 UTC 2024
Tue Feb 27 15:25:17 UTC 2024
Tue Feb 27 15:25:23 UTC 2024
Tue Feb 27 15:25:28 UTC 2024
```
```
vagrant@vagrant:~/kube/zad7$ kubectl delete deployment app-deployment
deployment.apps "app-deployment" deleted
vagrant@vagrant:~/kube/zad7$ kubectl delete pvc local-pvc
persistentvolumeclaim "local-pvc" deleted
vagrant@vagrant:~/kube/zad7$ kubectl describe pv local-pv
Name:            local-pv
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Released
Claim:           default/local-pvc
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /var/local-pv
    HostPathType:
Events:            <none>
```
- После удаления PVC и Deployment, состояние PV изменилось на "Released". Это означает, что PV больше не привязан к какому-либо PVC и доступен для повторного использования.
```
vagrant@vagrant:~/kube/zad7$ kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
vagrant   Ready    <none>   23d   v1.28.7
vagrant@vagrant:~/kube/zad7$ cd /var/local-pv
vagrant@vagrant:/var/local-pv$ ls
data.txt
vagrant@vagrant:/var/local-pv$ cat data.txt
Tue Feb 27 15:23:51 UTC 2024
Tue Feb 27 15:23:56 UTC 2024
Tue Feb 27 15:24:01 UTC 2024
Tue Feb 27 15:24:06 UTC 2024
Tue Feb 27 15:24:11 UTC 2024
Tue Feb 27 15:24:16 UTC 2024
Tue Feb 27 15:24:21 UTC 2024
Tue Feb 27 15:24:26 UTC 2024
Tue Feb 27 15:24:31 UTC 2024
Tue Feb 27 15:24:36 UTC 2024
Tue Feb 27 15:24:41 UTC 2024
Tue Feb 27 15:24:46 UTC 2024
```
```
vagrant@vagrant:/var/local-pv$ cd ~/kube/zad7
vagrant@vagrant:~/kube/zad7$ kubectl delete pv local-pv
persistentvolume "local-pv" deleted
vagrant@vagrant:~/kube/zad7$ kubectl describe pv local-pv
Error from server (NotFound): persistentvolumes "local-pv" not found
vagrant@vagrant:~/kube/zad7$ cd /var/local-pv
vagrant@vagrant:/var/local-pv$ ls
data.txt
vagrant@vagrant:/var/local-pv$ cat data.txt
Tue Feb 27 15:23:51 UTC 2024
Tue Feb 27 15:23:56 UTC 2024
Tue Feb 27 15:24:01 UTC 2024
Tue Feb 27 15:24:06 UTC 2024
Tue Feb 27 15:24:11 UTC 2024
Tue Feb 27 15:24:16 UTC 2024
Tue Feb 27 15:24:21 UTC 2024
Tue Feb 27 15:24:26 UTC 2024
Tue Feb 27 15:24:31 UTC 2024
Tue Feb 27 15:24:36 UTC 2024
Tue Feb 27 15:24:41 UTC 2024
Tue Feb 27 15:24:46 UTC 2024
```
- После удаления PV с помощью команды kubectl delete pv local-pv, файл data.txt остается на локальном диске ноды. Это происходит потому, что PV в Kubernetes управляет только ресурсами хранилища в кластере, но не управляет данными, которые могут храниться внутри него. Удаление PV просто освобождает ресурсы, занимаемые PV в кластере, но не затрагивает данные, которые были сохранены на узле хоста.
Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.
## Задание 2
- Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.[nts-pvc.yaml]()
- Включить и настроить NFS-сервер на MicroK8S.
- Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
- Продемонстрировать возможность чтения и записи файла изнутри пода.
- Предоставить манифесты, а также скриншоты или вывод необходимых команд.
```
vagrant@vagrant:~/kube/zad7$ sudo microk8s enable nfs 
Addon nfs was not found in any repository
To use the community maintained flavor enable the respective repository:
    microk8s enable community
