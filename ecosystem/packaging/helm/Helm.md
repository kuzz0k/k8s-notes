Пакетный менеджер для кубера: чарт это шаблоны манифестов, а `values.yaml` то, что в них подставляется. Свои values пишу только с тем, что меняю, остальное берется из дефолтов чарта.

### Установка

```
sudo snap install helm --classic
```

На k3s helm по умолчанию смотрит в `~/.kube/config`, а k3s кладет конфиг в `/etc/rancher/k3s/k3s.yaml`, поэтому копируем:

```
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
chmod 600 ~/.kube/config
```

### Чарт из OCI registry

Так ставил Envoy Gateway:

```
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.9.2 -n envoy-gateway-system --create-namespace
```

### Чарт из helm репы

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm show values prometheus-community/kube-prometheus-stack > default-values.yaml   # посмотреть все дефолты
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack --version 92.13.1 -n monitoring -f values.yaml
```

Руками так почти не ставил, мониторинг и логи ставятся чартами через [Argo CD](../../gitops/argocd/Argo%20CD.md#helm-чарт--values-из-своей-репы), версия чарта там в `targetRevision`.

Все values из проекта можете найти в [zero-to-cluster](https://github.com/kuzz0k/zero-to-cluster/tree/main/app-deploy/kubernetes/manifests/cluster).

### Что пригодилось в values

- Версию чарта всегда фиксировать, иначе при обновлении чарта поменяется все сразу
- Секреты в values не класть, у большинства чартов есть `existingSecret`, секрет создается руками:

```
grafana:
  admin:
    existingSecret: grafana-admin
    userKey: admin-user
    passwordKey: admin-password
```

- YAML якоря, чтобы не копировать одно и то же по всем компонентам:

```
prometheusOperator:
  affinity: &workers-only
    nodeAffinity:
      ...
prometheus:
  prometheusSpec:
    affinity: *workers-only
```

- Лишнее выключать через `enabled: false` (`replicas: 0`), на маленьких нодах это важно. Пример в [loki-values.yaml](../../observability/logging/loki-alloy/loki-values.yaml)
