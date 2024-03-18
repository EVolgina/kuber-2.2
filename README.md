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
- Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.[nts-pvc.yaml](https://github.com/EVolgina/kuber-2.2/blob/main/ntf-pvc.yaml) [sc]()
- Включить и настроить NFS-сервер на MicroK8S.
- Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.[deployment](https://github.com/EVolgina/kuber-2.2/blob/main/multitool-deployment.yaml). [pv](https://github.com/EVolgina/kuber-2.2/blob/main/pv-nfs.yaml)
- Продемонстрировать возможность чтения и записи файла изнутри пода.
- Предоставить манифесты, а также скриншоты или вывод необходимых команд.
```
vagrant@vagrant:~/kube/zad7$ dpkg -l | grep nfs-common
ii  nfs-common                            1:1.3.4-2.5ubuntu3.5              amd64        NFS support files common to client and server
vagrant@vagrant:~/kube/zad7$ microk8s enable nfs
Infer repository community for addon nfs
Addon community/nfs is already enabled
vagrant@vagrant:~/kube/zad7$ kubectl apply -f sc.yaml
storageclass.storage.k8s.io/nfs-csi created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f pv-nfs.yaml
persistentvolume/nfs-pv created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f nfs-pvc.yaml
persistentvolumeclaim/nfs created
vagrant@vagrant:~/kube/zad7$ kubectl apply -f multitool-deployment.yaml
deployment.apps/multitool created
vagrant@vagrant:~/kube/zad7$ kubectl get pv
NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                                                  STORAGECLASS   REASON   AGE
data-nfs-server-provisioner-0   1Gi        RWO            Retain           Terminating   nfs-server-provisioner/data-nfs-server-provisioner-0                           16d
nfs-pv                          1Gi        RWX            Retain           Bound         default/pvc-nfs                                        nfs-storage             49s
vagrant@vagrant:~/kube/zad7$ kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nfs   Bound    nfs-pv   1Gi        RWX            nfs-storage    46s
vagrant@vagrant:~/kube/zad7$ kubectl get pods
NAME                         READY   STATUS              RESTARTS   AGE
daemonset-5hmlm              1/1     Running             0          3d12h
multitool-8575674c98-jf2df   0/2     ContainerCreating   0          43s
vagrant@vagrant:~/kube/zad7$ kubectl get pods
NAME                         READY   STATUS              RESTARTS   AGE
daemonset-5hmlm              1/1     Running             0          3d12h
multitool-8575674c98-jf2df   0/2     ContainerCreating   0          72s
vagrant@vagrant:~/kube/zad7$ kubectl describe pod multitool-8575674c98-jf2df
Name:             multitool-8575674c98-jf2df
Namespace:        default
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Mon, 18 Mar 2024 05:13:53 +0000
Labels:           app=multitool
                  pod-template-hash=8575674c98
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/multitool-8575674c98
Containers:
  busybox:
    Container ID:
    Image:         busybox
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      while true; do date >> /srv/nfs/data.txt; sleep 5; done
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /srv/nfs from nfs-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qw895 (ro)
  multitool:
    Container ID:
    Image:          wbitt/network-multitool:latest
    Image ID:
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     10m
      memory:  20Mi
    Requests:
      cpu:     1m
      memory:  20Mi
    Environment:
      HTTP_PORT:  80
    Mounts:
      /srv/nfs from nfs-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qw895 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  nfs-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pvc-nfs
    ReadOnly:   false
  kube-api-access-qw895:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason       Age                From               Message
  ----     ------       ----               ----               -------
  Normal   Scheduled    92s                default-scheduler  Successfully assigned default/multitool-8575674c98-jf2df to vagrant
  Warning  FailedMount  21s (x8 over 87s)  kubelet            MountVolume.SetUp failed for volume "nfs-pv" : mount failed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 10.152.183.97:/srv/nfs /var/snap/microk8s/common/var/lib/kubelet/pods/cfce1acb-e084-43fe-9018-92f080c5d8de/volumes/kubernetes.io~nfs/nfs-pv
Output: mount.nfs: access denied by server while mounting 10.152.183.97:/srv/nfs
vagrant@vagrant:~/kube/zad7$ kubectl get sc
NAME          PROVISIONER         RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-storage   kubernetes.io/nfs   Delete          Immediate           false                  3d13h
vagrant@vagrant:~/kube/zad7$ kubectl describe sc nfs-storage
Name:            nfs-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"nfs-storage"},"mountOptions":["hard","nfsvers=4.1"],"parameters":{"path":"/srv/nfs","server":"10.152.183.97"},"provisioner":"kubernetes.io/nfs","reclaimPolicy":"Delete","volumeBindingMode":"Immediate"}

Provisioner:           kubernetes.io/nfs
Parameters:            path=/srv/nfs,server=10.152.183.97
AllowVolumeExpansion:  <unset>
MountOptions:
  hard
  nfsvers=4.1
ReclaimPolicy:      Delete
VolumeBindingMode:  Immediate
Events:             <none>
```
```
vagrant@vagrant:/srv/nfs$ kubectl cluster-info
Kubernetes control plane is running at https://10.0.2.15:16443
CoreDNS is running at https://10.0.2.15:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
vagrant@vagrant:/srv/nfs$ kubectl get services --all-namespaces
vagrant@vagrant:~/kube/zad7$ kubectl get services --all-namespaces
NAMESPACE                NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                     AGE
default                  kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                                                                                                     42d
kube-system              kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP                                                                                      42d
kube-system              metrics-server              ClusterIP   10.152.183.59    <none>        443/TCP                                                                                                     42d
kube-system              kubernetes-dashboard        ClusterIP   10.152.183.110   <none>        443/TCP                                                                                                     42d
kube-system              dashboard-metrics-scraper   ClusterIP   10.152.183.252   <none>        8000/TCP                                                                                                    42d
default                  my-service                  ClusterIP   10.152.183.48    <none>        9001/TCP,9002/TCP                                                                                           22d
default                  my-service1                 NodePort    10.152.183.118   <none>        80:30080/TCP                                                                                                22d
default                  svc-back                    ClusterIP   10.152.183.107   <none>        80/TCP                                                                                                      21d
default                  svc-front                   ClusterIP   10.152.183.79    <none>        80/TCP                                                                                                      21d
nfs-server-provisioner   nfs-server-provisioner      ClusterIP   10.152.183.97    <none>        2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,875/TCP,875/UDP,111/TCP,111/UDP,662/TCP,662/UDP   16d
vagrant@vagrant:~/kube/zad7$ kubectl get services --namespace nfs-server-provisioner
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                     AGE
nfs-server-provisioner   ClusterIP   10.152.183.97   <none>        2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,875/TCP,875/UDP,111/TCP,111/UDP,662/TCP,662/UDP   46h
```
-несколько раз пересоздавала pv pvc удаляла pod. Не могу найти причину почему pod не поднимается
```
```
