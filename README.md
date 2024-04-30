# Домашнее задание к занятию «Хранение в K8s. Часть 2»
- Цель задания: В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов
## Задание 1
- Что нужно сделать
- Создать Deployment приложения, использующего локальный PV, созданный вручную. [local-pv](https://github.com/EVolgina/kuber-2.2/blob/main/local-pv.yaml), [local-pvc](https://github.com/EVolgina/kuber-2.2/blob/main/local-pvc.yaml)
- Создать Deployment приложения, состоящего из контейнеров busybox и multitool. [Deployment](https://github.com/EVolgina/kuber-2.2/blob/main/app-deployment.yaml)
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
- Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.[nfs-pvc.yaml](https://github.com/EVolgina/kuber-2.2/blob/main/ntf-pvc.yaml) [sc](https://github.com/EVolgina/kuber-2.2/blob/main/sc.yaml)
- Включить и настроить NFS-сервер на MicroK8S.
- Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.[deployment](https://github.com/EVolgina/kuber-2.2/blob/main/multitool-deployment.yaml). [pv](https://github.com/EVolgina/kuber-2.2/blob/main/pv-nfs.yaml)
- Продемонстрировать возможность чтения и записи файла изнутри пода.
- Предоставить манифесты, а также скриншоты или вывод необходимых команд.
```
vagrant@vagrant:~/nf2$ newgrp microk8s
vagrant@vagrant:~/nf2$  sudo usermod -a -G microk8s vagrant
vagrant@vagrant:~/nf2$ sudo chown -R vagrant ~/.kube
vagrant@vagrant:~/nf2$ microk8s enable helm3
Infer repository core for addon helm3
Addon core/helm3 is already enabled
vagrant@vagrant:~/nf2$ microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
"csi-driver-nfs" has been added to your repositories
vagrant@vagrant:~/nf2$ microk8s helm3 repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "csi-driver-nfs" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
vagrant@vagrant:~/kube/zad7$ dpkg -l | grep nfs-common
ii  nfs-common                            1:1.3.4-2.5ubuntu3.6              amd64        NFS support files common to client and server
vagrant@vagrant:~/kube/zad7$ microk8s enable nfs
Infer repository community for addon nfs
Addon community/nfs is already enabled
vagrant@vagrant:~/nf2$ microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
>     --namespace kube-system \
>     --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
NAME: csi-driver-nfs
LAST DEPLOYED: Sun Apr 21 09:28:57 2024
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The CSI NFS Driver is getting deployed to your cluster.

To check CSI NFS Driver pods status, please run:

kubectl --namespace=kube-system get pods --selector="app.kubernetes.io/instance=csi-driver-nfs" --watch
vagrant@vagrant:~/nf2$ microk8s kubectl get csidrivers
NAME             ATTACHREQUIRED   PODINFOONMOUNT   STORAGECAPACITY   TOKENREQUESTS   REQUIRESREPUBLISH   MODES        AGE
nfs.csi.k8s.io   false            false            false             <unset>         false               Persistent   118s
vagrant@vagrant:~/kube/zad7$ kubectl apply -f sc.yaml
storageclass.storage.k8s.io/nfs-csi created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f pv-nfs.yaml
persistentvolume/nfs-pv created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f nfs-pvc.yaml
persistentvolumeclaim/nfs created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f multitool-deployment.yaml
deployment.apps/multitool created
vagrant@vagrant:~/nf2$ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
nfs-pv   1Gi        RWX            Retain           Bound    default/pvc-nfs   nfs-storage    <unset>                          85s
vagrant@vagrant:~/nf2$ kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-nfs   Bound    nfs-pv   1Gi        RWX            nfs-storage    <unset>                 78s
vagrant@vagrant:/srv/nfs$ microk8s kubectl get sc
NAME      PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-csi   nfs.csi.k8s.io   Delete          Immediate           false                  8m38s
vagrant@vagrant:/srv/nfs$ microk8s kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
multitool-8575674c98-p4ljl   2/2     Running   0          5m37s
```
```
vagrant@vagrant:/srv/nfs$ microk8s kubectl exec multitool-8575674c98-p4ljl -- ls /srv/nfs
Defaulted container "busybox" out of: busybox, multitool
data.txt
vagrant@vagrant:/srv/nfs$ microk8s kubectl exec multitool-8575674c98-p4ljl -- cat /srv/nfs/data.txt
Defaulted container "busybox" out of: busybox, multitool
Sun Apr 21 09:42:30 UTC 2024
Sun Apr 21 09:42:35 UTC 2024
Sun Apr 21 09:42:40 UTC 2024
Sun Apr 21 09:42:45 UTC 2024
Sun Apr 21 09:42:50 UTC 2024
Sun Apr 21 09:42:55 UTC 2024
Sun Apr 21 09:43:00 UTC 2024
Sun Apr 21 09:43:05 UTC 2024
Sun Apr 21 09:43:10 UTC 2024
Sun Apr 21 09:43:15 UTC 2024
Sun Apr 21 09:43:20 UTC 2024
Sun Apr 21 09:43:25 UTC 2024
Sun Apr 21 09:43:31 UTC 2024
Sun Apr 21 09:43:36 UTC 2024
Sun Apr 21 09:43:41 UTC 2024
Sun Apr 21 09:43:46 UTC 2024
Sun Apr 21 09:43:51 UTC 2024
Sun Apr 21 09:43:56 UTC 2024
Sun Apr 21 09:44:01 UTC 2024
Sun Apr 21 09:44:06 UTC 2024

vagrant@vagrant:~/nf2$ microk8s kubectl exec multitool-8575674c98-p4ljl -- touch /srv/nfs/new_file.txt
Defaulted container "busybox" out of: busybox, multitool
vagrant@vagrant:~/nf2$ microk8s kubectl exec multitool-8575674c98-p4ljl -- ls /srv/nfs/new_file.txt
Defaulted container "busybox" out of: busybox, multitool
/srv/nfs/new_file.txt
vagrant@vagrant:~/nf2$ microk8s kubectl exec multitool-8575674c98-p4ljl -- sh -c 'echo "Hello, World!" > /srv/nfs/new_file.txt'
Defaulted container "busybox" out of: busybox, multitool
vagrant@vagrant:~/nf2$ microk8s kubectl exec multitool-8575674c98-p4ljl -- cat /srv/nfs/new_file.txt
Defaulted container "busybox" out of: busybox, multitool
Hello, World!
```
