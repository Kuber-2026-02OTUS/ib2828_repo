### установки Consul
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install consul hashicorp/consul -n consul --create-namespace -f consul-values.yaml


### установка Vault:
helm install vault hashicorp/vault -n vault --create-namespace -f vault-values.yaml


helm upgrade --install vault hashicorp/vault   --namespace vault   --values vault-values.yaml   --version 0.34.0

## Инициализация Vault и распечатывание

kubectl exec -n vault vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > vault-keys.json

Получение unseal key и root token:
UNSEAL_KEY=$(cat vault-keys.json | jq -r ".unseal_keys_b64[]")
ROOT_TOKEN=$(cat vault-keys.json | jq -r ".root_token")


Распечатывание всех подов Vault
kubectl exec -n vault vault-0 -- vault operator unseal $UNSEAL_KEY
kubectl exec -n vault vault-1 -- vault operator unseal $UNSEAL_KEY
kubectl exec -n vault vault-2 -- vault operator unseal $UNSEAL_KEY

Получение unseal key и root token:
UNSEAL_KEY=$(cat vault-keys.json | jq -r ".unseal_keys_b64[]")
ROOT_TOKEN=$(cat vault-keys.json | jq -r ".root_token")

Распечатывание всех подов Vault:
kubectl exec -n vault vault-0 -- vault operator unseal $UNSEAL_KEY
kubectl exec -n vault vault-1 -- vault operator unseal $UNSEAL_KEY
kubectl exec -n vault vault-2 -- vault operator unseal $UNSEAL_KEY


### Создание секрета в Vault

Логин в Vault:
kubectl exec -n vault vault-0 -- vault login $ROOT_TOKEN

Включение KV секретов:
kubectl exec -n vault vault-0 -- vault secrets enable -path=otus kv-v2

Создание секрета otus/cred:
kubectl exec -n vault vault-0 -- vault kv put otus/cred username=otus password=otus


### ServiceAccount и ClusterRoleBinding

Создание сервисного аккаунта sa-vault:
kubectl apply -f kubernetes/sa-vault.yaml

Создание ClusterRoleBinding **auth-delegator**:
kubectl apply -f kubernetes/crb-auth-delegator.yaml


### Настройка Kubernetes Auth в Vault

Включение метода kubernetes в движке auth:
kubectl exec -n vault vault-0 -- vault auth enable kubernetes

Получение JWT токена и CA сертификата:
kubectl exec -n vault vault-0 -- sh -c 'vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt'


### Создание политики Vault

Применение политики **otus-policy**:
cat otus-policy.hcl | kubectl exec -n vault vault-0 -i -- vault policy write otus-policy -


### Создание роли в Vault

kubectl exec -n vault vault-0 -- vault write auth/kubernetes/role/otus \
  bound_service_account_names=vault-auth \
  bound_service_account_namespaces=vault \
  policies=otus-policy \
  ttl=24h

### Установка External Secrets Operator

Команда установки External Secrets Operator:
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace -f terraform/charts/external-secrets-values.yaml

helm upgrade --install external-secrets \
  external-secrets/external-secrets \
  -n external-secrets \
  --create-namespace \
  --set installCRDs=true

### Создание SecretStore

kubectl apply -f kubernetes/secretstore.yaml

### Создание ExternalSecret

kubectl apply -f kubernetes/external-secrets.yaml




####################################
#####################################
####################################




```


#### 14. Установка Ingress

В качестве балансировщика будем использовать Ingress-Nginx:
```bash
kubectl apply -f kubernetes/ingress-nginx.yaml
```

Добавить строку в /etc/hosts (добавляем внешний ip адрес балансировщика YandexCloud):
```bash
echo $(kubectl get service/ingress-nginx-controller -n ingress-nginx -o "jsonpath={.status.loadBalancer.ingress[0].ip}") homework.otus vault.homework.otus consul.homework.otus | sudo tee -a /etc/hosts
```

В браузере можно войти в веб-версию Vault пройдя по ссылке:
```
http://vault.homework.otus
```
![alt text](<pics/Screenshot from 2025-11-03 20-30-53.png>)

![alt text](<pics/Screenshot from 2025-11-03 20-31-04.png>) 

![alt text](<pics/Screenshot from 2025-11-03 20-31-35.png>) 

![alt text](<pics/Screenshot from 2025-11-03 20-31-46.png>)

И также можно войти в веб-версию Consul пройдя по ссылке:
```
http://consul.homework.otus
```
![alt text](<pics/Screenshot from 2025-11-03 20-28-23.png>)

#### Проверка результата

Проверка созданного секрета
```bash
kubectl get secret -n vault otus-cred -o yaml
```

Декодирование и проверка значений
```bash
kubectl get secret -n vault otus-cred -o jsonpath='{.data.username}' | base64 --decode ; echo
kubectl get secret -n vault otus-cred -o jsonpath='{.data.password}' | base64 --decode ; echo
```

Все необходимые файлы конфигурации и команды предоставлены. После выполнения всех шагов должна быть полностью рабочая система с Vault, использующая Consul в качестве бэкенда хранения, и External Secrets Operator для синхронизации секретов в Kubernetes.

