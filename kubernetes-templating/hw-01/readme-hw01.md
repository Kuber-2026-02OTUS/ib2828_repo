Проверка на ошибки
helm template homework ./hw-01/webserver   --set replicaCount=2 --set ingress.host=test.local
Установка
helm install homework ./hw-01/webserver   --set replicaCount=2 --set ingress.host=test.local
