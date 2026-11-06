Логи собирает Grafana Alloy, хранит Loki, смотрю в той же Grafana из [kube-prometheus-stack](<../../metrics/kube-prometheus-stack/Prometheus и Grafana.md>). Promtail больше не развивается (EOL), поэтому Alloy.

Установку в проекте можете найти здесь: [zero-to-cluster → Loki](https://github.com/kuzz0k/zero-to-cluster/blob/main/docs/setup/Monitoring.md#loki), values в [logging](https://github.com/kuzz0k/zero-to-cluster/tree/main/app-deploy/kubernetes/manifests/cluster/logging).

Файлы:
- [application.yaml](application.yaml) - Argo CD Application `logging`: чарты `loki` и `alloy` + values из своей репы
- [loki-values.yaml](loki-values.yaml)
- [alloy-values.yaml](alloy-values.yaml)

Ставится в тот же ns `monitoring`, namespace создает приложение `monitoring`, поэтому оно должно стоять раньше.

```
kubectl apply -f application.yaml
kubectl -n argocd get application logging
kubectl -n monitoring get pods -o wide | grep -E "loki|alloy"
```

`loki-0` на воркере, `alloy-*` по одному на каждой ноде.

### Loki

Одним подом (SingleBinary). Режим по умолчанию с отдельными read/write/backend и memcached слишком тяжелый для маленьких нод, все лишнее выключено:

```
deploymentMode: SingleBinary

singleBinary:
  replicas: 1

read:
  replicas: 0
write:
  replicas: 0
backend:
  replicas: 0

gateway:
  enabled: false
chunksCache:
  enabled: false
resultsCache:
  enabled: false
```

Хранение на NFS 10Gi, логи 7 дней (retention работает только с включенным compactor):

```
  limits_config:
    retention_period: 168h
    discover_log_levels: true      # Loki сам определяет detected_level
    volume_enabled: true

  compactor:
    retention_enabled: true
    delete_request_store: filesystem

  pattern_ingester:
    enabled: true                  # вкладка Patterns в Drilldown → Logs
```

Проверка:

```
kubectl -n monitoring port-forward svc/loki 3100
curl localhost:3100/ready
curl localhost:3100/loki/api/v1/labels
```

### Alloy

DaemonSet на каждой ноде, читает логи прямо с диска ноды `/var/log/pods` (hostPath). Стримить логи через API кубера не стал, это лишняя нагрузка на API server на маленькой control ноде.

```
controller:
  type: daemonset

alloy:
  mounts:
    varlog: true
```

Конфиг в `alloy.configMap.content`, по шагам:

1. `discovery.kubernetes` - находит поды только своей ноды:

```
      discovery.kubernetes "pods" {
        role = "pod"
        selectors {
          role  = "pod"
          field = "spec.nodeName=" + sys.env("HOSTNAME")
        }
      }
```

2. `discovery.relabel` - вешает лейблы `namespace`, `pod`, `container`, `node`, `app` и собирает путь до файла лога `/var/log/pods/*<uid>/<container>/*.log`
3. `local.file_match` + `loki.source.file` - читают файлы
4. `loki.process` - разбор и фильтрация:

```
        stage.cri {}     # формат логов containerd

        # успешные пробы и скрейпы Prometheus, иначе забивают все остальное
        stage.drop {
          expression          = "\"GET /(health|healthz|ready|metrics)[^ ]* HTTP/[0-9.]+\" 200"
          drop_counter_reason = "probes_and_scrapes"
        }

        # вырезать токен Telegram бота из логов Alertmanager
        stage.replace {
          expression = "bot[0-9]+:([A-Za-z0-9_-]+)"
          replace    = "<redacted>"
        }
```

5. `loki.write` - отправка в `http://loki:3100/loki/api/v1/push`

Если логов нет, первым делом смотрим Alloy:

```
kubectl -n monitoring logs ds/alloy -c alloy --tail=50
```

У Alloy есть UI с графом компонентов, видно сколько файлов читает и что отправляет. Сервис ведет на случайный под, для конкретной ноды port-forward на сам под:

```
kubectl -n monitoring port-forward svc/alloy 12345
```

### В Grafana

Loki добавлен datasource в values kube-prometheus-stack (`additionalDataSources`). Дашборд `Kubernetes - Logs` ([k8s-logs.json](../../metrics/kube-prometheus-stack/extras/dashboards/k8s-logs.json)) с фильтрами по namespace, app, pod и уровню. Значения фильтров берутся из лейблов Loki, новые ns и приложения появляются сами.
