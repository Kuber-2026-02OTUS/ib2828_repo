
### Создаём StorageClass
kubectl apply -f vault-local-storage.yaml

### Создаём PV
kubectl apply -f vault-local-pv.yaml

### Добавляем репозиторий
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

### Устанавливаем Vault
helm install vault hashicorp/vault -n vault -f vault-values.yaml

### Инициализируем Vault
kubectl exec -it -n vault vault-0 -- vault operator init

### Распечатываем Vault
kubectl exec -it -n vault vault-0 -- vault operator unseal

### Подключаем остальные инстансы и распечатываем
kubectl exec -n vault vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -it -n vault vault-0 -- vault operator unseal
kubectl exec -n vault vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -it -n vault vault-0 -- vault operator unseal

### Создаем сервисный аккаунт
kubectl apply -f sa-vault.yaml

### Создаем сервисный ClusterRoleBinding
Создание ClusterRoleBinding **auth-delegator**:
kubectl apply -f crb.yaml

### Включаем авторизацию kubernetes в движке auth
kubectl exec -n vault vault-0 -- -i -- sh -c 'export VAULT_TOKEN="VAULT_TOKEN"; vault auth enable kubernetes'

### Применяем политики **otus-policy**:
cat otus-policy.hcl | kubectl exec -n vault vault-0 -i -- sh -c 'export VAULT_TOKEN="VAULT_TOKEN"; vault policy write otus-policy -'
