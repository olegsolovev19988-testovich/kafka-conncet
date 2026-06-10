# jdbc-asterisk-source

JDBC Source коннектор для таблиц Asterisk CDR.

## Зависимости

- Плагин: `jdbc` (собирается в `cluster/kafka-connect.yaml`)
- Kubernetes Secret `jdbc-asterisk-source` в namespace `kafka-connect`

## Секреты

Пути Vault — в [secrets/vault-reference.yaml](secrets/vault-reference.yaml).

## Применение

```bash
kubectl apply -f connector.yaml -n kafka-connect
```
