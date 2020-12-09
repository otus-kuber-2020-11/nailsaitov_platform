# Выполнено ДЗ №2

 - [ ] Основное ДЗ
 - [ ] Задание со *
 - [ ] Задание со **

## В процессе сделано:
 - Установлен kind и создан локальный кластер k8s
 - Создан и применен манифест ReplicaSet - frontend-replicaset.yaml
 - Собран и помещен в Docker Hub образ с двумя тегами v0.0.1 и v0.0.2 для paymentService
 - Созданы и применены манифесты ReplicaSet(paymentservice-replicaset.yaml) & Deployment(paymentservice-deployment.yaml), с применением разных кейсов развертывания
 - Реализован сценарий blue-green - paymentservice-deployment-bg.yaml
 - Реализован сценарий Reverse Rolling Update - paymentservice-deployment-reverse.yaml
 - Создан и применен манифест DaemonSet для node-exporter - node-exporter-daemonset.yaml

## Решение Д/З №2

- Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?
``Ответ: ReplicaSet Controller не поддерживает напрямую Rolling update, тем самым, если запущенно указанное количество pod-ов, на этом его полномочия заканчиваются.``

- Deployment | Задание со ⭐

``В результате получили два манифеста:
  paymentservice-deployment-bg.yaml
  paymentservice-deployment-reverse.yaml``

  Применили манифест с readinessProbe

- DaemonSet | Задание со ⭐
Написал манифест node-exporter-daemonset.yaml для развертывания DaemonSet с Node Exporter

- DaemonSet | Задание с ⭐⭐
Для того, чтобы Node-Exporter развернулся на всех мастерах и нодах в кластере добавил сделующие параметры в манифесте:
```   spec:
        tolerations:
        - operator: "Exists"
```
## Как проверить работоспособность:
Выполнить команды:
 
`kubectl port-forward <имя любого pod в DaemonSet> 9100:9100`
`curl localhost:9100/metrics`
