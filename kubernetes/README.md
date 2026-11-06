# Kubernetes

Тут база по кубу. Конспекты переносил из обсидиана, рядом с ними лежат манифесты, которые гонял на своем k3s кластере. Папки пронумерованы примерно в том порядке, в котором я все это проходил.

## 00-basics

С чего начинал, как вообще устроен кубер

- [Примерная цепочка в k8s](<00-basics/Примерная цепочка в k8s.md>)
- [Элементы k8s](<00-basics/Элементы k8s.md>)
- [Namespace](00-basics/Namespace.md)
- [Подключение Lens](<00-basics/Подключение Lens.md>)

Еще там лежит `Архитектура.canvas`, его открывать в обсидиане

## 01-workloads

Pod, Deployment, StatefulSet, DaemonSet, Job

- [Pod](01-workloads/pod/Pod.md)
- [Init контейнеры](<01-workloads/pod/Init контейнеры.md>)
- [best practice для манифестов](<01-workloads/pod/best practice для манифестов.md>)
- [Deployment](01-workloads/deployment/Deployment.md)
- [Restart, rollout и стратегии обновления](<01-workloads/deployment/Restart (rollout, стратегии).md>)
- [StatefulSet](01-workloads/statefulset/StatefulSet.md), в [psql](01-workloads/statefulset/psql) поднимал постгрес через StatefulSet
- [DaemonSet](01-workloads/daemonset/DaemonSet.md)
- [Job и CronJob](<01-workloads/job-cronjob/Job и CronJob.md>)

## 02-scheduling

Как раскидывать поды по нодам

- [Affinity, Taints, NodeSelector](<02-scheduling/affinity/Affinity, Taints, NodeSelector.md>)
- [PriorityClass](02-scheduling/priority-class/PriorityClass.md)

## 03-config

Как прокидывать конфиги и данные в поды

- [ConfigMap](03-config/configmap/ConfigMap.md)
- [Secret](03-config/secret/Secret.md)
- [Downward API](<03-config/downward-api/Downward API.md>)
- [Projected Volumes](<03-config/projected-volume/Projected Volumes.md>)

## 04-storage

Тома. PV делал и руками, и через nfs provisioner

- [EmptyDir](04-storage/emptydir/EmptyDir.md)
- [HostPath](04-storage/hostpath/HostPath.md)
- [PV, основы](<04-storage/persistent-volume/1. PV — основы.md>)
- [Ручное создание PV](<04-storage/persistent-volume/2. Ручное создание PV.md>)
- [Динамическое создание](<04-storage/persistent-volume/3. Динамическое создание.md>)
- [Примеры использования](<04-storage/persistent-volume/4. Примеры использования.md>)

## 05-network

Самая большая часть

- [Kubernetes network model](<05-network/Kubernetes network model.md>)
- [Service](05-network/service/Service.md)
- DNS: [настройки pod](<05-network/dns/Настройки pod.md>), [DNS у kubespray](<05-network/dns/DNS у kubespray.md>), [траблшутинг](05-network/dns/Траблшутинг.md)
- [Ingress](05-network/ingress/Ingress.md)
- [MetalLB](05-network/metallb/MetalLB.md)

Gateway API

- [Теория](<05-network/gateway-api/Gateway API — теория.md>)
- [TLS сертификаты](<05-network/gateway-api/tls/TLS сертификаты.md>)
- [Traefik + MetalLB](<05-network/gateway-api/traefik-metallb/Traefik + MetalLB.md>)
- [Envoy + MetalLB](<05-network/gateway-api/envoy-metallb/Envoy + MetalLB.md>)

## 06-rbac

- [RBAC](06-rbac/RBAC.md)
- [Role](06-rbac/Role.md)
- [Создание сертификата для пользователя](<06-rbac/Создание сертификата для пользователя.md>)

## 07-resource-quota

- [ResourceQuota](07-resource-quota/ResourceQuota.md), там же LimitRange
