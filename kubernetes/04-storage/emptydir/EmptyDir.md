Используется для предоставления временного хранилища поду. Они создаются при запуске пода и удаляются при его завершении. Данные в таких томах доступны всем контейнерам пода и могут использоваться для обмена информацией между ними.

### Основные характеристики 

- Создается при запуске пода и удаляется при его завершении
- Изначально пустой
- Доступен всем контейнерам в поде
- Может использовать различные типы носителей (диск, SSD, память)
- Не сохраняет данные после удаления пода

### Типипчные сценарии использования

- **Обмен данными между контейнерами**
- **Временное хранилище**
- **Подготовка данных с помощью Init Container**

Не стоит использовать для БД!!

### Создание EmptyDir

Создается в определении пода в спецификации `volumes`:

```
spec:
  containers:
  - name: emptydir-container
    image: busybox
    command: ["/bin/sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
    volumeMounts:
    - name: emptydir-volume
      mountPath: /usr/share/emptydir
  volumes:
  - name: emptydir-volume
    emptyDir: {}
```

По умолчанию будет создаваться `EmptyDir` на диске, но если необходимо его создать в памяти:

```
volumes:
- name: emptydir-volume
  emptyDir: 
    medium: Memory
```

**Внимание:** При использовании памяти том будет потреблять оперативную память, и если лимиты будут превышены, то придет OOMKilled

### Управление размером emptyDir

```
volumes:
- name: emptydir-volume
  emptyDir: 
    medium: Memory
    sizeLimit: 100Mi
```

Если контейнер попытается записать в `/usr/share/emptydir` больше 100Mi, под будет выселен (evicted) с узла

### Пример использования EmptyDir для обмена данными между контейнерами

В примере будет два контейнера: `producer` для создания записей и `consumer` для их чтения

Посмотрим логи

```
kubectl -n work logs -f emptydir-shared-pod -c consumer
```

##### Как тома emptyDir отображаются на файловую систему сервера

Для этого нужно узнать:
- Сервер, где запущен под
- UID пода

```
kubectl -n work get pod app-deployment-6c5f5b8dd6-2jm5f -o custom-columns=NAME:.metadata.name,UID:.metadata.uid,NODE:.spec.nodeName
```

Далее подключимся к ноде по SSH и перейдем по:

```
ls -l /var/lib/kubelet/pods/{UID}/volumes/kubernetes.io~empty-dir/shared-volume/
```

### Пример использования EmptyDir с InitContainer

Следующий пример демонстриурет, как `InitContainer` может подготовить конфигурационные файлы для основного контейнера, используя `emptyDir` в качестве общего хранилища

Создадим `ConfigMap`

Далее создадим `Deployment`. `InitContainer` (config-init) прочитает шаблон из `ConfigMap`, заменит переменные-плейсхолдеры на реальные значения из переменных окружения и запишет итоговый файл в `emptyDir` том. Основной контейнер (`app-container`) затем сможет использовать этот готовый файл

Простыми словами:
Берется `ConfigMap` и используется как темплейт, в нем заменяются поля, и темплейт записывается уже в `EmptyDir`. Сам `ConfigMap` не изменяется!!

```
kubectl -n work get pods -l app.kubernetes.io/name=app-deployment
```

Посмотрим содержимое сгенерированного файла:

```
kubectl -n work exec -it \
	$(kubectl -n work get pods -l app.kubernetes.io/name=app-deployment -o jsonpath='{.items[0].metadata.name}') \
	-c app-container -- cat /etc/app-config/app-config.txt
```

