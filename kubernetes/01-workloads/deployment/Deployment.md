В основном предназначен для `stateless` приложений, то есть приложений, которые не сохраняют состояние.
```
spec:
	replicas: 3
	selector:
		matchLabels:
			app: *nginx-app
			version: *nginx-version
	template:
```

- replicas - количество подов
- matchLabels - позволяет отличие поды для deployment
- template - далее идет темлейт самого пода

### ReplicaSet

**ReplicaSet** — это контроллер Kubernetes, который поддерживает заданное количество одинаковых Pod. Если Pod удаляется или завершается с ошибкой, ReplicaSet автоматически создаёт новый.

На практике ReplicaSet редко создают вручную — обычно им управляет Deployment.

---

## Связь Deployment → ReplicaSet → Pod

```

- **Deployment** управляет ReplicaSet.
    
- **ReplicaSet** управляет Pod.
    
- **Pod** содержит один или несколько контейнеров.
    

При изменении Deployment Kubernetes создаёт новый ReplicaSet, который постепенно заменяет старый (Rolling Update).

---

# Как появляется ReplicaSet

При выполнении:

```bash
kubectl apply -f deployment.yaml
```

создаётся **Deployment**, а затем **Deployment Controller** автоматически создаёт соответствующий ReplicaSet.

ReplicaSet, в свою очередь, создаёт необходимое количество Pod.

Поэтому после создания Deployment команда

```bash
kubectl -n work get all
```

показывает примерно следующее:

```text
NAME                                   READY
pod/my-first-deployment-xxxxx          1/1

NAME                                         DESIRED
replicaset.apps/my-first-deployment-xxxxx    3

NAME                              READY
deployment.apps/my-first-deployment 3/3
```

Хотя ReplicaSet в манифесте не описывался, он был автоматически создан Deployment Controller.

---

# Команда kubectl get all

```bash
kubectl -n work get all
```

Показывает основные ресурсы пространства имён:

- Pods
    
- Services
    
- Deployments
    
- ReplicaSets
    
- DaemonSets
    
- StatefulSets
    
- Jobs
    
- CronJobs (в зависимости от версии Kubernetes)
    

Эта команда удобна для быстрого просмотра состояния приложения.

---

# Просмотр selector ReplicaSet

```bash
kubectl -n work get rs my-first-deployment-28359 \
  -o jsonpath='{.spec.selector}' | jq
```

Что делает команда:

1. Получает объект ReplicaSet.
    
2. Извлекает поле `.spec.selector`.
    
3. Передаёт результат в `jq` для красивого форматирования.
    

Пример вывода:

```json
{
  "matchLabels": {
    "app": "nginx",
    "pod-template-hash": "28359"
  }
}
```

**Selector** определяет, какими Pod управляет ReplicaSet. Если Pod имеет соответствующие метки (`labels`), ReplicaSet считает его своим.

---

# OwnerReferences

Каждый объект Kubernetes может иметь владельца (Owner).

Посмотреть владельца можно командой:

```bash
kubectl -n work get rs my-first-deployment-28359 \
  -o jsonpath='{.metadata.ownerReferences}'
```

Пример:

```json
[
  {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "name": "my-first-deployment",
    "controller": true
  }
]
```

Поле `ownerReferences` показывает, какой ресурс создал данный объект.

В данном случае ReplicaSet принадлежит Deployment.
