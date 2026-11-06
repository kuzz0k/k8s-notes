ResoqurceQuota определяет ограничения на namespace.

- Количество объектов, создаваемых в namespace, по типу.
- Общий объем вычислительных ресурсов.
- Объём и количество дискового пространства для volumes.

Квоты работаю следующим образом:

1. Создается ResourceQuota в namespace.
2. Пользователи создают объекты в namespace. Система отслеживает, не превышают ли запрошенные ресурсы лимиты, описанные в квотах.
3. Если создание нового ресурса превышает квоту, API сервер возвращает 403-ю ошибку (FORBIDDEN) с сообщением о том, какие квоты были превышены.

Если в namespace включена квота на вычислительные ресурсы: cpu и memory. В создаваемых пользователем ресурсах **должны быть явно описаны лимиты.** Если лимиты не описаны, система отклонит создание новых ресурсов. Для избегания подобной ситуации администратор должен устанавливать LimitRanger на namespace.

```
kubectl -n q-test describe quota
```

При добавлении квот, рекомендуется определить LimitRange, он выставляет минимальные ограничения по умолчанию, если не указывать при создании подов

```
---
apiVersion: v1
kind: LimitRange
metadata:
  name: q-test-lr
  namespace: q-test
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: "0.1"
    defaultRequest:
      memory: 256Mi
      cpu: "0.1"
    max:
      memory: 2Gi
      cpu: "2"
    type: Container
```

## Типы квот

### Вычислительные

- limits.cpu
- limits.memory
- requests.cpu
- requests.memory
### Хранения

- requests.storage
- persistentvolumeclaims
- [storage-class-name].storageclass.storage.k8s.io/requests.storage
- [storage-class-name].storageclass.storage.k8s.io/persistentvolumeclaims

### Количество объектов

- configmaps
- pods
- replicationcontrollers
- resourcequotas
- services
- services.loadbalancers
- services.nodeports
- secrets

Пример ResourceQuota:

```
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: q-test-quota
  namespace: q-test
spec:
  hard:
    # Вычислительные квоты
    requests.cpu: "0.2"
    requests.memory: 200Mi
    limits.cpu: "2"
    limits.memory: 2Gi
    # Квоты на количество объектов
    pods: "2"
    services: "2"
    services.nodeports: "0"
    # Квоты хранения
    persistentvolumeclaims: "2"
    managed-nfs-storage.storageclass.storage.k8s.io/requests.storage: 8Gi
    managed-nfs-storage.storageclass.storage.k8s.io/persistentvolumeclaims: "1"
```