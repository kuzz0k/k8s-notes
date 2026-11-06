[Документация](https://kubectl.docs.kubernetes.io/references/kustomize/)

Встроен в kubectl, шаблонов нет: берет обычные манифесты из `resources` и патчит их по правилам из `kustomization.yaml`.

```
kubectl kustomize ./backend    # посмотреть, что получится
kubectl apply -k ./backend
```

Argo CD сам определяет Kustomize, если в папке есть `kustomization.yaml`.

Как это используется в проекте: манифесты фронта и бэка в [deploy-app](https://github.com/kuzz0k/zero-to-cluster/tree/main/app-deploy/kubernetes/manifests/deploy-app), CI, который меняет тег, в [backend-ci.yml](https://github.com/kuzz0k/zero-to-cluster/blob/main/backend/.gitea/workflows/backend-ci.yml).

### Тег образа

Использовал, чтобы CI менял одну строку, а Deployment оставался нетронутым, [пример](examples/image-tag):

```
resources:
  - deployment.yaml

images:
  - name: 192.168.1.205:3000/user1/backend
    newTag: "1.0.1"
```

В `deployment.yaml` тег может быть любой, Kustomize подменит его во всех контейнерах с этим образом, включая init-контейнер с миграцией. Поэтому миграции всегда идут из той же версии, что и приложение.

Тег из CI меняется через sed, подробнее в [Argo CD](../../gitops/argocd/Argo%20CD.md#связка-с-ci)

### configMapGenerator

Так складывал дашборды Grafana из json в ConfigMap, [kustomization.yaml](../../observability/metrics/kube-prometheus-stack/extras/kustomization.yaml):

```
generatorOptions:
  disableNameSuffixHash: true      # без хеша в имени
  labels:
    grafana_dashboard: "1"         # навешивается на все сгенерированные ConfigMap

configMapGenerator:
  - name: dashboard-node-use
    namespace: monitoring
    files:
      - dashboards/node-use.json
    options:
      annotations:
        grafana_folder: Cluster
```

По умолчанию к имени ConfigMap добавляется хеш от содержимого, чтобы поды перезапускались при изменении. Для дашбордов это не нужно, sidecar Grafana и так их подхватывает.
