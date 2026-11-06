# Ecosystem

Тут то, что ставится в кластер поверх самого кубера.

База по самому куберу лежит в [kubernetes](../kubernetes).

## gitops

- [Argo CD](<gitops/argocd/Argo CD.md>), там же связка с CI в Gitea

## packaging

- [Helm](packaging/helm/Helm.md)
- [Kustomize](packaging/kustomize/Kustomize.md)

## observability

- метрики: [Prometheus и Grafana](<observability/metrics/kube-prometheus-stack/Prometheus и Grafana.md>) (kube-prometheus-stack)
- алерты: [Alertmanager](observability/alerting/alertmanager/Alertmanager.md), уведомления в Telegram
- логи: [Loki и Alloy](<observability/logging/loki-alloy/Loki и Alloy.md>)
