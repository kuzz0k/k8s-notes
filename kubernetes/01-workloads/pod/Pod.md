Pod - наименьший компонент, содержащий коллекцию контейнеров. 
##### Создание Pod
```
kubectl apply -f pod.yaml
```
##### Просмотр доступных подов для данного namespace
```
kubectl -n work get pods
```
##### Посмотреть описание пода
```
kubectl describe -n work pod my-pod | less
kubectl get pod my-pod -n work -o yaml | less - в формате yml после создания
```
##### Удалить Pod
```
kubectl delete -n work pod my-pod
```
##### Посмотреть логи
```
kubectl -n work logs my-pod
```
конкретного контейнера
```
kubectl -n work logs my-pod -c init-myservice
```



### Состояния Pod в Kubernetes

```
Pending → ContainerCreating → Running → Succeeded / Failed
                          ↘
                    CrashLoopBackOff
```

- **Pending** — Pod создан, ожидает назначения на ноду или загрузки образов.
- **ContainerCreating** — kubelet создаёт контейнеры и монтирует тома.
- **Running** — все контейнеры запущены и работают.
- **Succeeded** — контейнеры успешно завершили работу (код выхода 0).
- **Failed** — контейнер завершился с ошибкой.
- **CrashLoopBackOff** — контейнер постоянно падает, Kubernetes пытается его перезапускать.
- **Terminating** — Pod удаляется и ожидает завершения контейнеров.
Также стоит смотреть на раздел `Events`, он показывает события которые произошли в кластере, там можно посмотреть почему контейнер не запустился.

# Состояния контейнеров в Kubernetes

```
Waiting → Running → Terminated
      ↘
 CrashLoopBackOff
```

- **Waiting** — контейнер ожидает запуска (например, скачивается образ, ожидаются зависимости или секреты).
- **Running** — контейнер успешно запущен и выполняется.
- **Terminated** — контейнер завершил работу (успешно или с ошибкой).
- **CrashLoopBackOff** — контейнер завершается с ошибкой, Kubernetes пытается перезапустить его с увеличивающейся задержкой.

## Причины состояния Waiting

- **ContainerCreating** — создаётся контейнер, монтируются тома, настраивается сеть.
- **ImagePullBackOff** — не удалось скачать образ, повторные попытки с задержкой.
- **ErrImagePull** — ошибка при загрузке образа.
- **CreateContainerConfigError** — ошибка в конфигурации контейнера (например, отсутствует Secret или ConfigMap).
- **CreateContainerError** — ошибка при создании контейнера.

## Причины состояния Terminated

- **Completed** — контейнер завершился успешно (код выхода `0`).
- **Error** — контейнер завершился с ошибкой (код выхода ≠ `0`).
- **OOMKilled** — контейнер был остановлен из-за нехватки памяти (Out Of Memory).
- **Killed** — контейнер был принудительно остановлен.

##### Якоря

В yaml формате есть якоря, позволяющие создать переменную `&` и использовать ее дальше в конфиге `*`:
```
metadata:
	name: nginx-deployment
	namespace: work
	labels:
		app: &nginx-app nginx
		version: &nginx-version 1.27.5
spec:
	replicas: 3
	selector:
		matchLabels:
			app: *nginx-app
			version: *nginx-version
```

### Selector

```
spec:
  selector:
    matchLabels:
      app: nginx
      version: 1.27.5
```

Он говорит Deployment’у:
«Ты управляешь только теми Pod’ами, у которых есть labels `app=nginx` и `version=1.27.5`»