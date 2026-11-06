Более распространенный механизм, существует:
- `Node Affinity` - определяет предпочтение размещения подов на нодах кластера
- `Pod Affinity` - определяет правила размещения подов рядом с другими подами
- `Pod Anti-affinity` - определяет правила, не разрешающие размещения подов на нодах, где уже запущены такие поды

Самый распространненый вариант использования - это `Pod Anti-affinity`. Когда нам необходимо запретить размещение двух подов одной управляющей конструкции `Deployment` или `StatefulSet` на одной ноде кластера. Такое размещение необходимо в основном для отказоустойчивости приложения

У `podAntiAffinity` есть два варианта:
- `requiredDuringSchedulingIgnoredDuringExecution` - обязательный к исполнению, если не удается подобрать ноду с данным условием - под создаваться не будет
- `preferredDuringSchedulingIgnoredDuringExecution` - мягкий вариант, если нода будет не найдена - все равно запустится под на любой ноде

```
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values: ['test-deployment', 'test-deployment-2']
              - key: app.kubernetes.io/version
                operator: In
                values: ['v0.0.1']
          topologyKey: kubernetes.io/hostname
```

В данном примере будут создаваться поды только на тех нодах, где нет подов с name - `['test-deployment', 'test-deployment-2']` и version - `['v0.0.1']`

На самом деле, метка `kubernetes.io/hostname` по умолчанию устанавливается на все ноды кластера.

Запустим манифест и заметим, что поднялись два пода, а один в состоянии Pending. Это связано с тем что control нода заражена, а tolerations мы не прописывали.

Посмотрим подробнее:

```
kubectl -n work events --for pod/test-deployment-fahdj
```

```

```


### NodeAffinity

`NodeAffinty` позволяет запускать поды на определенных нодах, например по меткам.

Добавим метки:

```
kubectl label nodes worker1 topology.kubernetes.io/zone=Africa
kubectl label nodes worker2 topology.kubernetes.io/zone=Africa
kubectl label nodes control topology.kubernetes.io/zone=Australia
```

И добавим `nodeAffinity` в манифест:

```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/zone
              operator: In
              values: ['Africa']
```

Теперь посмотрим на какие ноды приземлился под:

```
kubectl -n work get pods -o='custom-columns=Name:.metadata.name,Node:.spec.nodeName'
```
### weight

Вес позволяет расставить весовой коэфициент для предпочтения расставления подов по нодам. То есть чем больше `weight` - тем больше преоритет.

```
affinity:
  nodeAffinity:
    prefferedDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: topology.kubernetes.io/zone
              operator: In
              values: ['Africa']
      - weight: 10
        preference:
          matchExpressions:
            - key: topology.kubernetes.io/zone
              operator: In
              values: ['Australia']
```

### Pod Topology Spread Constraints

Нужен для того, чтобы поды равномерно распределялись между нодами, можно описать какая разница в количестве допустима. [Документация](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)

```
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLables:
          app.kubernetes.io/name: *name
          app.kubernetes.io/version: *version
          app.kubernetes.io/instance: *instance
      nodeAffinityPolicy: Ingore
      nodeTaintsPolicy: Honor
```

- `maxSkew` — максимальная разница количества подов между доменами топологии.
- `topologyKey` — метка на ноде кластера, которая используется для определения доменов топологии.
- `whenUnsatisfiable` — что делать с подом, если он не соответствует ограничению.
  - `DoNotSchedule` — (по умолчанию) запрещает планировщику запускать под на ноде.
  - `ScheduleAnyway` — разрешает запускать под на ноде.
- `labelSelector` — определяет список меток подов, попадающих под это правило.
- `nodeAffinityPolicy` — определяет будут ли учитываться `nodeAffinity`/`nodeSelector` пода при расчёте неравномерности распределения пода.
  - `Honor` — (по умолчанию) в расчёт включаются только ноды, соответствующие `nodeAffinity`/`nodeSelector`.
  - `Ignore` — в расчёты включены все ноды.
- `nodeTaintsPolicy` — аналогично `nodeAffinityPolicy`, только учитываются `Taints`.
  - `Honor` — включаются ноды без установленных `Taints`, а также ноды, для которых у пода есть `Toleration`.
  - `Ignore` — (по умолчанию) в расчёты включены все ноды.

Чтобы проверить распределение подов, их можно запускать вручную:

```
kubectl -n work scale --replicas=2 deployment test-deployment
```

### NodeName

В DaemonSet уже затрагивали тему распределения подов по нодам, но в поде можно явно указать где ему создаваться.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
  nodeName: worker1
```

# Зараженные ноды (Taints и Tolerations)

Чтобы посмотреть заразы на нодах можно использовать следующую команду:

```
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

Зараза состоит из трех значений `key=[value]:Effect`
Ключ, значение и эффект. Ключ может быть каким угодно, но лучше использовать [список ключей](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#taints-and-tolerations)
Эффекты бывают:
- `NoSchedule` - запрещает планирование под на ноде, поды запущенные до применения taints не будут удалены
- `NoExecute` - запрещает планирование под на ноде, поды запущенные до применения taints будут удалены
- `PreferNoSchedule` - планировщик пытается не размещать поды на ноде (мягкая версия NoSchedule)

Чтобы все-таки запустить под на зараженной ноде необходимо добавить toleration:

```
spec:
  tolerations:
    - key: "node-role.kubernetes.io/master"
      operator: "Exists"
      effect: "NoSchedule"
```

Теперь посмотрим что выведет

```
kubectl -n work get pods -o wide
```

### Заражение ноды

Это может делать для того чтобы остальные приложения игнорировали данную ноду, а ваше толерантное запускалось именно там. Демонсет сам смотрит на taint и при их изменении сам поднимает поды или удаляет их.

```
kubectl taint nodes worker.local test-taint=:NoExecute
```

Чтобы удалить taint необходимо добавить `-` в конце:

```
kubectl taint nodes worker.local test-taint=:NoExecute-
```

# NodeSelector

NodeSelector позволяет определить ноды кластера куда будет ставиться под, причем в качестве условия мы должны ставить label на ноды кластера
Чтобы поставить метку на ноду:

```
kubectl label nodes worker.local special=ds-only
```

А посмотреть метки на нодах:

```
kubectl get nodes --show-labels
```

Добавим `NodeSelector` в поды:

```
spec:
  nodeSelector:
    special: ds-only
```