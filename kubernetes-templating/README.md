# Выполнено ДЗ №7

## В процессе сделано:
 - [x] Основное ДЗ
 - [x] Задание со *

Была выполнена регистрация в GCP, создан аккаунт и настроено подключение. 

Создаем кластер k8s: 
``
gcloud container clusters get-credentials otus-hw-cluster --zone us-central1-c --project infra-301118
``

Кластер создан:
```
kubectl get nodes -o wide
NAME                                             STATUS   ROLES    AGE   VERSION             INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-otus-hw-cluster-default-pool-372d7335-1pj7   Ready    <none>   18h   v1.16.15-gke.4901   10.128.0.2    35.225.154.14 Container-Optimized OS from Google   4.19.112+    docker://19.3.1
gke-otus-hw-cluster-default-pool-372d7335-qdrg   Ready    <none>   18h   v1.16.15-gke.4901   10.128.0.4    35.239.63.153   Container-Optimized OS from Google   4.19.112+    docker://19.3.1
gke-otus-hw-cluster-default-pool-372d7335-qs30   Ready    <none>   18h   v1.16.15-gke.4901   10.128.0.3    35.232.13.224   Container-Optimized OS from Google   4.19.112+    docker://19.3.1
```
### Устанавливаем готовые Helm charts со следующими сервисами:
- [nginx-ingress](https://github.com/kubernetes/ingress-nginx) сервис, обеспечивающий доступ к публичным ресурсам кластера
- [cert-manager](https://github.com/jetstack/cert-manager/tree/master/deploy/charts/cert-manager) сервис, позволяющий динамически генерировать Let's Encrypt сертификаты для ingress ресурсов
- [chartmuseum](https://github.com/helm/charts/tree/master/stable/chartmuseum) специализированный репозиторий для хранения helm charts
- [harbor](https://github.com/goharbor/harbor-helm) хранилище артефактов общего назначения (Docker Registry),поддерживающее helm charts

Для начала нам необходимо установить Helm 3 на локальную машину
По команде helm version получаем следующий результат:
```
version.BuildInfo{Version:"v3.2.1", GitCommit:"fe51cd1e31e6a202cba7dead9552a6d418ded79a", GitTreeState:"clean", GoVersion:"go1.13.10"}
```
По умолчанию в Helm 3 не установлен репозиторий stable, устанавливаем:
```
helm repo add stable https://charts.helm.sh/stable
```
Создадим namespace и release nginx-ingress
```
kubectl create ns nginx-ingress
helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress --version=1.41.3
```
### cert-manager
Добавляем репозиторий: 
```
helm repo add jetstack https://charts.jetstack.io
```
Устанавливаем cert-manager:
```
kubectl create ns cert-manager
helm upgrade --install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.16.0 \
  --set installCRDs=true
```
### chartmuseum
Кастомизируем установку chartmuseum

Реализуем [values.yaml](https://github.com/otus-kuber-2020-11/nailsaitov_platform/blob/main/kubernetes-templating/chartmuseum/values.yaml) 

Устанавливаем chartmuseum:
```
kubectl create ns chartmuseum
helm upgrade --install chartmuseum stable/chartmuseum --wait \
--namespace=chartmuseum \
--version=2.13.2 \
-f kubernetes-templating/chartmuseum/values.yaml
```
Проверяем, что release chartmuseum установился:
```
helm ls -n chartmuseum
```
Критерий успешности установки: 
- Chartmuseum доступен по URL https://chartmuseum.34.68.47.24.nip.io
- Сертификат для данного URL валиден

### Установите harbor в кластер с использованием helm3
Для этого: 
- Реализуем файл [values.yaml](https://github.com/otus-kuber-2020-11/nailsaitov_platform/blob/main/kubernetes-templating/harbor/values.yaml)
- Установил harbor (собственного домена нет, тем самым не получилось поднять harbor с сертификатом, https://harbor.34.68.47.24.xip.io/ не защищен!!!:
```
helm repo add harbor https://helm.goharbor.io
helm repo update
kubectl create ns harbor
helm upgrade --install harbor harbor/harbor --wait \
--namespace=harbor \
--version=1.1.2 \
-f kubernetes-templating/harbor/values.yaml
```
Реквизиты по умолчанию: admin/Harbor12345

Критерий успешности установки: 
- Chartmuseum доступен по URL https://harbor.34.68.47.24.xip.io

### Создаем свой helm chart
#### Мы рассмотривали третий вариант. Взяли готовые манифесты и подготовили их к релизу на разные окружения.
Использовали демо-приложение [hipster-shop](https://github.com/GoogleCloudPlatform/microservices-demo), представляющее собой типичный набор микросервисов.

Стандартными средствами helm инициализировали структуру директории с содержимым будущего helm chart
```
helm create kubernetes-templating/hipster-shop
```

Helm chart уже готов, устанавливаем его:
```
kubectl create ns hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```
Так как все микросервисы устанавливаются из одного файла  all-hipstershop.yaml, а это делаем helm chart не практичным, будем разделять, и первым идет frontend

Создаем заготовку:
```
helm create kubernetes-templating/frontend
```
Аналогично чарту hipster-shop удаляем файл values.yaml и файлы в директории templates, создаваемые по умолчанию. 
Выделим из файла all-hipster-shop.yaml манифесты для установки микросервиса frontend. В директории templates чарта frontend создаем файлы: 
- deployment.yaml, service.yaml, ingress.yaml

После того, как вынесли описание deployment и service для frontend из файла all-hipster-shop.yaml переустанавливаем chart hipster-shop и проверяем,, что доступ к UI пропал и таких ресурсов больше нет.

Установавливаем chart frontend в namespace hipster-shop и проверяем что доступ к UI вновь появился:
```
helm upgrade --install frontend kubernetes-templating/frontend --namespace hipster-shop
```

#### Пришло время шаблонизировать наш chart frontend
Создаем файл values.yaml
Далее в манифесте deployment.yaml надо указать, переменную для tag
- image: gcr.io/google-samples/microservices-demo/frontend:{{ .Values.image.tag }}

Аналогичным образом шаблонизируем следующие параметры frontend chart:
- Количество реплик в deployment
- Port, targetPort и NodePort в service

Теперь наш frontend стал немного похож на настоящий helm chart
Добавьим chart frontend как зависимость
```
dependencies:
  - name: frontend
    version: 0.1.0
    repository: "file://../frontend" - ссылается на /frontend
```
Обновим зависимости:
```
helm dep update kubernetes-templating/hipster-shop
```
В директории kubernetes-templating/hipster-shop/charts появился архив frontend-0.1.0.tgz содержащий chart frontend определенной версии и добавленный в chart hipster-shop как зависимость. Обновим release hipster-shop и убедимсяь, что ресурсы frontend вновь созданы.
```
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```
##Создаем свой helm chart | Задание со⭐ 

Провели аналогичную процедуру с Redis, как делали с frontend.

