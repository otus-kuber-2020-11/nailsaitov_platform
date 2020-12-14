# Выполнено ДЗ №3

 - [x] Основное ДЗ

## В процессе сделано:

 - task01
1. Создан Service Account bob, и присвоена ему роль admin в рамках всего кластера
2. Создан`` Service Account dave без доступа к кластеру

 - task02
1. Создан Namespace prometheus
2. Создан Service Account carol в этом Namespace
3. Дана возможность всем Service Account в Namespace prometheus делать get, list, watch в отношении Pods всего кластера

 - task03
1. Создан Namespace dev
2. Создан Service Account jane в Namespace dev
3. Jane дана роль admin в рамках Namespace dev
4. Создан Service Account ken в Namespace dev
5. Ken дана роль view в рамках Namespace dev
