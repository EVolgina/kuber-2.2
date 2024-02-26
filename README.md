# Домашнее задание к занятию «Хранение в K8s. Часть 2»
- Цель задания: В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов
## Задание 1
- Что нужно сделать
- Создать Deployment приложения, использующего локальный PV, созданный вручную. [local-pv](), [local-pvc]()
- Создать Deployment приложения, состоящего из контейнеров busybox и multitool. []()
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
```
