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
- Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.[nts-pvc.yaml](https://github.com/EVolgina/kuber-2.2/blob/main/ntf-pvc.yaml)
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
vagrant@vagrant:~/kube/zad7$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs    Bound    nfs-pv   1Gi        RWX            nfs-storage    59s
vagrant@vagrant:~/kube/zad7$ kubectl get pv
NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                  STORAGECLASS   REASON   AGE
data-nfs-server-provisioner-0   1Gi        RWO            Retain           Bound    nfs-server-provisioner/data-nfs-server-provisioner-0                           11d
nfs-pv                          1Gi        RWX            Retain           Bound    default/nfs                                            nfs-storage             75s
vagrant@vagrant:~/kube/zad7$ kubectl get sc
NAME                          PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
nfs-storage                   kubernetes.io/nfs                      Retain          Immediate              false                  11d
nfs                           cluster.local/nfs-server-provisioner   Delete          Immediate              true                   11d
microk8s-hostpath (default)   microk8s.io/hostpath                   Delete          WaitForFirstConsumer   false                  9d
nfs-csi                       nfs.csi.k8s.io                         Delete          Immediate              false                  2m
vagrant@vagrant:~/kube/zad7$ kubectl get pods -A
NAMESPACE                NAME                                         READY   STATUS              RESTARTS         AGE
kube-system              csi-nfs-node-5kdvx                           3/3     Running             4 (8d ago)       12d
nfs-server-provisioner   nfs-server-provisioner-0                     1/1     Running             1 (8d ago)       11d
kube-system              kubernetes-dashboard-54b48fbf9-b4wq6         1/1     Running             33 (8d ago)      37d
kube-system              dashboard-metrics-scraper-5657497c4c-6748c   1/1     Running             20 (8d ago)      37d
kube-system              calico-kube-controllers-76c98cc5-t7mfl       1/1     Running             8 (8d ago)       17d
kube-system              calico-node-sj2lr                            1/1     Running             13 (8d ago)      17d
ingress                  nginx-ingress-microk8s-controller-csnhr      1/1     Running             1 (8d ago)       16d
default                  daemonset-l869k                              1/1     Running             1 (8d ago)       16d
kube-system              coredns-864597b5fd-5flhg                     1/1     Running             20 (8d ago)      37d
kube-system              metrics-server-848968bdcd-7n9jf              1/1     Running             46 (5d4h ago)    37d
kube-system              hostpath-provisioner-7df77bc496-rnj86        1/1     Running             11 (90m ago)     9d
kube-system              csi-nfs-controller-d96ccb59c-spqvp           4/4     Running             20 (83m ago)     12d
default                  multitool-pod                                0/1     ImagePullBackOff    376 (102m ago)   17d
default                  multitool-7dd685cf9d-kx9b2                   0/2     ContainerCreating   0                109s
```
```
vagrant@vagrant:/srv/nfs$ kubectl cluster-info
Kubernetes control plane is running at https://10.0.2.15:16443
CoreDNS is running at https://10.0.2.15:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
vagrant@vagrant:/srv/nfs$ kubectl get services --all-namespaces
vagrant@vagrant:~/kube/zad7$ kubectl get services --all-namespaces
NAMESPACE                NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                     AGE
default                  kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                                                                                                     28d
kube-system              kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP                                                                                      28d
kube-system              metrics-server              ClusterIP   10.152.183.59    <none>        443/TCP                                                                                                     28d
kube-system              kubernetes-dashboard        ClusterIP   10.152.183.110   <none>        443/TCP                                                                                                     28d
kube-system              dashboard-metrics-scraper   ClusterIP   10.152.183.252   <none>        8000/TCP                                                                                                    28d
default                  my-service                  ClusterIP   10.152.183.48    <none>        9001/TCP,9002/TCP                                                                                           8d
default                  my-service1                 NodePort    10.152.183.118   <none>        80:30080/TCP                                                                                                8d
default                  svc-back                    ClusterIP   10.152.183.107   <none>        80/TCP                                                                                                      7d7h
default                  svc-front                   ClusterIP   10.152.183.79    <none>        80/TCP                                                                                                      7d7h
nfs-server-provisioner   nfs-server-provisioner      ClusterIP   10.152.183.97    <none>        2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,875/TCP,875/UDP,111/TCP,111/UDP,662/TCP,662/UDP   46h
vagrant@vagrant:~/kube/zad7$ kubectl get services --namespace nfs-server-provisioner
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                     AGE
nfs-server-provisioner   ClusterIP   10.152.183.97   <none>        2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,875/TCP,875/UDP,111/TCP,111/UDP,662/TCP,662/UDP   46h
```
-несколько раз пересоздавала pv pvc удаляла pod. Не могу найти причину почему pod не поднимается
```
vagrant@vagrant:~/kube/zad7$ kubectl describe pods multitool-77497c5659-ftvkq
Name:             multitool-77497c5659-ftvkq
Namespace:        default
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Tue, 12 Mar 2024 16:47:04 +0000
Labels:           app=multitool
                  pod-template-hash=77497c5659
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/multitool-77497c5659
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
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kb5bq (ro)
  multitool:
    Container ID:
    Image:          wbitt/network-multitool:latest
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /srv/nfs from nfs-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kb5bq (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  nfs-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  nfs
    ReadOnly:   false
  kube-api-access-kb5bq:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason       Age               From               Message
  ----     ------       ----              ----               -------
  Normal   Scheduled    78s               default-scheduler  Successfully assigned default/multitool-77497c5659-ftvkq to vagrant
  Warning  FailedMount  5s (x8 over 78s)  kubelet            MountVolume.SetUp failed for volume "nfs-pv" : mount failed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 10.152.183.97:/srv/nfs /var/snap/microk8s/common/var/lib/kubelet/pods/d6407e9f-013a-40b7-ac69-ac8ac4ae6f4d/volumes/kubernetes.io~nfs/nfs-pv
Output: mount.nfs: access denied by server while mounting 10.152.183.97:/srv/nfs
```
