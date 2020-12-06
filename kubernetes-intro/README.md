### Выполненное Д/З №1

 - [x] Основное ДЗ
 - [x] Задание со *

### В процессе домашнего задания было сделано:
- Настроено локальное окружение MicroK8s на LInux Ubuntu 20.04
- Написан Dockerfile и собран образ с веб серевером nginx на alpine и помещен на Docker Hub [https://hub.docker.com/repository/docker/nailsaitov/otus.hw1.kubernetes-intro.web]
- Склонирован и собран собственный образ для frontend и помещен на Docker Hub [https://hub.docker.com/repository/docker/nailsaitov/otus.hw1.kubernetes-intro.frontend]

---

### Решение задания №1

- Разберитесь почему все pod в namespace kube-system восстановились после удаления. 

Статические поды контролируются демоном kubelet, в то время как остальные находятся под контролем компонетов control plane. (https://kubernetes.io/docs/concepts/workloads/pods/)

Описания подов хранятся в следующей директории:
```
$ ls -la /etc/kubernetes/manifests
total 20
drwxr-xr-x 2 root root 4096 Jul 23 23:01 .
drwxr-xr-x 4 root root 4096 Oct  9 12:01 ..
-rw------- 1 root root 3011 Jun  9 15:28 kube-apiserver.yaml
-rw------- 1 root root 2773 Sep 23  2019 kube-controller-manager.yaml
-rw------- 1 root root  990 Sep 23  2019 kube-scheduler.yamlger.yaml
-rw------- 1 root root 1.1K Aug  9 10:28 kube-scheduler.yaml
```
CoreDNS pod является деплойментом:
```
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns                     1/1     1            1           17
```

### Hipster Shop | Задание со ⭐

- Выясните причину, по которой pod frontend находится в статусе Error

По причине того, что не были определены env: 
```
$kubectl logs pod/frontend
  panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
```
В манифест были добавлены следующие переменные:

```kubectl set env pod/frontend --list
# Pod frontend, container frontend
PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
PORT=8080
PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
CURRENCY_SERVICE_ADDR=currencyservice:7000
CART_SERVICE_ADDR=cartservice:7070
RECOMMENDATION_SERVICE_ADDR=recommendationservice:8080
SHIPPING_SERVICE_ADDR=shippingservice:50051
CHECKOUT_SERVICE_ADDR=checkoutservice:5050
AD_SERVICE_ADDR=adservice:9555
ENV_PLATFORM=gcp```
