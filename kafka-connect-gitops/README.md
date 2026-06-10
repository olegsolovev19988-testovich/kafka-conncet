# Kafka Connect GitOps (Strimzi)

Декларативное развёртывание Kafka Connect и коннекторов через [Strimzi](https://strimzi.io/).

## Структура репозитория

```
kafka-connect-gitops/
├── README.md
├── kustomization.yaml              # cluster + connectors (опционально)
├── cluster/                        # Kafka Connect cluster (инфраструктура)
│   ├── kafka-connect.yaml          # KafkaConnect CR + build.plugins
│   ├── metrics-config.yaml
│   └── kustomization.yaml
└── kafka-connect-connectors/       # Коннекторы
    ├── connectors/
    ├── templates/
    └── README.md
```

## Требования

- Kubernetes 1.25+
- [Strimzi Operator](https://strimzi.io/docs/operators/latest/deploying.html) в namespace `kafka-connect`
- Docker registry для образа Connect (Strimzi `spec.build`)
- Kafka bootstrap: `10.0.2.9:9092` (измените в `cluster/kafka-connect.yaml`)

## Установка Strimzi Operator

```bash
kubectl create namespace kafka-connect
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka-connect' -n kafka-connect
kubectl wait --for=condition=Ready pod -l name=strimzi-cluster-operator -n kafka-connect --timeout=300s
```

## Деплой

### 1. Connect cluster (плагины + 2 реплики)

Замените `registry.example.com/kafka-connect:3.8.0` на свой registry в `cluster/kafka-connect.yaml`.

```bash
kubectl apply -k cluster/ -n kafka-connect
kubectl get kafkaconnect connect-cluster -n kafka-connect -w
```

Strimzi соберёт образ с плагинами:
- `debezium-postgres`
- `jdbc`

### 2. Коннекторы

```bash
kubectl apply -k kafka-connect-connectors/connectors/ -n kafka-connect
```

## Проверка

```bash
# Кластер
kubectl get kafkaconnect,pods -n kafka-connect -l strimzi.io/cluster=connect-cluster

# Плагины (port-forward к Connect REST API)
kubectl port-forward svc/connect-cluster-connect-api 8083:8083 -n kafka-connect
curl -s http://localhost:8083/connector-plugins | jq '.[].class'

# Коннекторы
kubectl get kafkaconnector -n kafka-connect
```

## Argo CD

Рекомендуется два Application:

| App | Path | Описание |
|-----|------|----------|
| `kafka-connect-cluster` | `cluster/` | KafkaConnect, редкие изменения |
| `kafka-connect-connectors` | `kafka-connect-connectors/connectors/` | Коннекторы, частые изменения |

## Добавление плагина

В `cluster/kafka-connect.yaml` → `spec.build.plugins`:

```yaml
- name: my-plugin
  artifacts:
    - type: zip
      url: https://example.com/plugin.zip
```

После apply Strimzi пересоберёт образ и сделает rolling update.

## Добавление коннектора

См. [kafka-connect-connectors/README.md](kafka-connect-connectors/README.md).
