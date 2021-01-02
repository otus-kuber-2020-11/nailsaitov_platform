# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 - Развернут StatefulSet c - локальным S3 хранилищем

**Применение StatefulSet**
Закомитил конфигурацию под именем minio-statefulset.yaml

В результате применения конфигурации произошло следующее:
- Запустился под с MinIO
- Создался PVC
- Динамически создался PV на этом PVC с помощью дефолотного StorageClass

**Применение Headless Service**
Закомитил конфигурацию под именем minio-headlessservice.yaml.

**Проверка работы MinIO**
 - kubectl get statefulsets
 - kubectl get pods
 - kubectl get pvc
 - kubectl get pv
 - kubectl describe <resource> <resource_name>

### Задание со ⭐
- Был создал secret.yaml и добавлены параметры в манифест minio-statefulset.yaml

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
