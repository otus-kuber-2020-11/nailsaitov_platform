# Выполнено ДЗ №8

 - [x] Основное ДЗ
 - [x] Задание со 🐍

## В процессе сделано:
 - Написан CustomResource и CustomResourceDefinition для mysql оператора 
 - Написана часть логики mysql оператора при помощи python KOPF
 - Собран образ и сделан деплой оператора

### Подготовка
- Был поднят kubernetes кластер в minikube
- Также были установлены различные зависимости для python

### Создаем CRD и CR
Для создания pod с MySQL оператору, создали следующие манифесты:
- Создан CustomResource [cr.yaml](https://github.com/otus-kuber-2020-11/nailsaitov_platform/blob/main/kubernetes-operators/deploy/cr.yaml)
- Создан CustomResourceDefinition [crd.yaml](https://github.com/otus-kuber-2020-11/nailsaitov_platform/blob/main/kubernetes-operators/deploy/crd.yaml)

### Validation
Добавили в спецификацию CRD ( spec ) параметры validation

### 🐍 MySQL контроллер
В папке kubernetes-operators/build создал файл mysql-operator.py. Для написания контроллера использовали фреймворк kopf с добавлением необходимых зависимых библиотек.
```
import kopf
import yaml
import kubernetes
import time
from jinja2 import Environment, FileSystemLoader
```
В дирректории kubernetes-operators/build/templates создали различные шаблоны.
```
mysql-deployment.yml.j2
mysql-service.yml.j2
mysql-pv.yml.j2
mysql-pvc.yml.j2
backup-pv.yml.j2
backup-pvc.yml.j2
backup-job.yml.j2
restore-job.yml.j2
```

Произвели доработки контроллера
- Добавили функцию, для обработки Jinja шаблонов и преобразования YAML в JSON
- Добавили декоратор
- Добавили в декоратор рендер шаблонов

Произвели промежуточную проверку, что обрабатываться события при оздании cr.yaml:
```
kopf run mysql-operator.py
```
Вывод:
```
[2021-01-09 22:57:34,059] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'mysql_on_create' succeeded.
[2021-01-09 22:57:34,059] kopf.objects         [INFO    ] [default/mysql-instance] All handlers succeeded for creation.
```
**_Вопрос: почему объект создался, хотя мы создали CR, до того, как запустили контроллер?_**
На момент запуска контроллера CR уже был создан, но события о нём продолжают периодически приходить, тем самым сработала функция и создались ресурсы.

Сделали сделующие доработки:
- Для удаления ресурсов при удалении CR, сделали deployment,svc,pv,pvc дочерними ресурсами к mysql.
- Добавили создание PV и PVC для бэкапов.
- Добавили функцию delete_success_jobs.
- Добавлили функция wait_until_job_end
- Вызов созданных функций добавлен в delete_object_make_backup
- Добавили генерацию из шаблонов для restore-job
- Добавили восстановление из бэкапов.

### Запускаем оператор (из директории build):
```
kopf run mysql-operator.py
```
### Создаем CR:
```
kubectl apply -f deploy/cr.yaml
```
### Проверяем что появились pvc:
```
kubectl get pvc
NAME                        STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound     pvc-71b946d4-a7d6-4dbf-ae32-828d55b255a9       10Gi       RWO           standard       84m
mysql-instance-pvc          Bound     pvc-01099bd4-7753-463c-9c5b-ea6bc16856c7       10Gi       RWO           standard       84m
```
Заполнение созданной БД
```
MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
kubectl exec -it "${MYSQLPOD}" -- mysql -u root  -potuspassword -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database
kubectl exec -it "${MYSQLPOD}" -- mysql -potuspassword  -e "INSERT INTO test ( id, name )VALUES ( null, 'some data' );" otus-database
kubectl exec -it "${MYSQLPOD}" -- mysql -potuspassword -e "INSERT INTO test ( id, name )VALUES ( null, 'some data-2' );" otus-database
```
Просмотр таблицы:
```
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```
После того, как мы убедились, что наш контроллер работает, теперь нужно собрать Docker образ и запушить его в dockerhub:
```
docker build -t nailsaitov/mysql-operator:1.0.0 .
docker push hub.docker.com/nailsaitov/mysql-operator:1.0.0
docker push nailsaitov/mysql-operator:1.0.0
```

Повторная провека показала, успешную работоспособность контроллера.
