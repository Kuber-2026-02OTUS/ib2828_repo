#######################################################################
#######################################################################
#######################################################################
Создаем пространство имен homework
kubectl apply -f namespace.yaml

Регистрируем CustomResourceDefinition mysqls.otus.homework в кластере
kubectl apply -f crd.yaml

Создаем ServiceAccount, ClusterRole и ClusterRoleBinding
kubectl apply -f rbac.yaml

Разворачиваем сам оператор
kubectl apply -f deployment.yaml

Создаем PersistentVolume для базы данных
kubectl apply -f pv.yaml

Создаем кастомный ресурс MySQL
kubectl apply -f mysql.yaml

Создаем сервис
kubectl apply -f service.yaml


#######################################################################
#######################################################################
#######################################################################
Проверяем CRD
kubectl get crd mysqls.otus.homework

Проверяем оператор
kubectl get pods -n homework


Проверяем деплоймент
kubectl get deployments -n homework

Проверяем сервис
kubectl get svc -n homework

Проверяем хранилище на диске
kubectl get pv -n homework

Проверяем кастомный ресурс
kubectl get mysql -n homework

Удаляем кастомный ресурс
kubectl delete mysql homework-mysql -n homework

