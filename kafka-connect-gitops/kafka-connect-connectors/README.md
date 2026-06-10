# Kafka Connect Connectors

GitOps-репозиторий коннекторов для Strimzi Kafka Connect cluster `connect-cluster`.

## Структура

```
kafka-connect-connectors/
├── connectors/                    # Экземпляры коннекторов (по одной папке на коннектор)
│   ├── debezium-opws-postgres/
│   │   ├── connector.yaml         # KafkaConnector CR
│   │   ├── README.md
│   │   └── secrets/               # Только ссылки на Vault
│   ├── jdbc-asterisk-source/
│   └── ...
├── templates/                     # Базовые шаблоны для новых коннекторов
│   ├── debezium-postgres-base.yaml
│   └── jdbc-source-base.yaml
└── README.md
```

Инфраструктура Connect (кластер, плагины, replicas) — в соседней папке [`../cluster/`](../cluster/).

## Добавление нового коннектора

1. Скопируйте шаблон из `templates/`:
   ```bash
   cp templates/debezium-postgres-base.yaml connectors/my-new-connector/connector.yaml
   ```
2. Создайте `connectors/my-new-connector/secrets/vault-reference.yaml` с путями Vault.
3. Добавьте `connector.yaml` в `connectors/kustomization.yaml`.
4. Синхронизируйте K8s Secret из Vault (External Secrets / Vault Agent).
5. Примените:
   ```bash
   kubectl apply -k connectors/ -n kafka-connect
   ```

## Секреты

- В git хранятся **только** ссылки на Vault (`secrets/vault-reference.yaml`).
- В `connector.yaml` используется синтаксис Strimzi:
  ```yaml
  database.password: ${secrets:kafka-connect/my-secret:password}
  ```
- Целевой Kubernetes Secret создаётся вне этого репозитория (CI, External Secrets Operator).

## Проверка

```bash
kubectl get kafkaconnector -n kafka-connect
kubectl describe kafkaconnector <name> -n kafka-connect
```
