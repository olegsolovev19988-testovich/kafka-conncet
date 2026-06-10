# debezium-opws-postgres

CDC-коннектор Debezium для PostgreSQL базы OPWS.

## Зависимости

- Kafka Connect cluster: `connect-cluster` (Strimzi)
- Плагин: `debezium-postgres` (собирается в `cluster/kafka-connect.yaml`)
- Kubernetes Secret `debezium-opws-postgres` в namespace `kafka-connect`

## Секреты

Пути Vault — в [secrets/vault-reference.yaml](secrets/vault-reference.yaml).

## PostgreSQL prerequisites

```sql
ALTER SYSTEM SET wal_level = logical;
-- перезапуск PostgreSQL

CREATE PUBLICATION dbz_opws_publication FOR TABLE public.orders, public.customers;
```

## Применение

```bash
kubectl apply -f connector.yaml -n kafka-connect
```

## Проверка

```bash
kubectl get kafkaconnector debezium-opws-postgres -n kafka-connect
kubectl describe kafkaconnector debezium-opws-postgres -n kafka-connect
```
