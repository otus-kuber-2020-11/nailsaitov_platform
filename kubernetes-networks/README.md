# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:

> План работы: Работа с тестовым веб-приложением

- Добавление проверок Pod
- Создание объекта Deployment
- Добавление сервисов в кластер ( ClusterIP )
- Включение режима балансировки IPVS

> План работы: Доступ к приложению извне кластера

- становка MetalLB в Layer2-режиме
- Добавление сервиса LoadBalancer
- Установка Ingress-контроллера и прокси ingress-nginx
- Создание правил Ingress

**Добавление проверок Pod**

В описание были добавлены 2 вида проверок: readinessProbe & livenessProbe для web-pod.yaml

**Создание Deployment**
Далее создали deployment web-deploy.yaml c измененными конфигурациями readinessProbe = 8000 и replicas = 3

**Создание Service**
Для того, чтобы наше приложение было доступно внутри кластера нам потребовался объект типа Service . Начали с
распространенного типа сервисов - ClusterIP. Создали сервис web-svc-cip.yaml 

**Включение IPVS**
Включили IPVS для kube-proxy , исправив ConfigMap выполнив команду:
`kubectl --namespace kube-system edit configmap/kube-proxy`

### Работа с LoadBalancer и Ingress
**Установка MetalLB**
Произвели установку с помощью команд:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```
Создал манифест metallb-config.yaml 

## Как проверить работоспособность:
 - Произвести проверку через браузер по адресу http://172.17.255.1/

Задание со ⭐ | DNS через MetalLB
Манифесты находится в подкаталоге ./coredns

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
