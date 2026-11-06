### Volumes

Приложение в контейнере может создавать файлы в файловой системе. Но при удалении контейнера и создании нового, эти файлы будут удаляться.

Создавать файлы в внутри файловой системы контейнера не самая разумная идея, поэтому если планируется работать с файлами, лучше подключить тома.

### Тома в k8s

Условно тома в k8s можно разделить на:
- `Ephemeral volume` - тома удаляемые после завершения работы пода.
- `Persistent volumes` - тома доступные после удаления пода

##### Ephemeral volume

Создаются kubelet при запуске контейнеров пода. Хранятся непосредственно на ноде.

### ConfigMap

Обычно используется для подключения env и конфигов к контейнеру подов

##### env

Переменные окружения можно передавать напрямую в манифесте:

```
env:
  - name: TEST_ENV
    key: test-value
```

Посмотрим env в контейнере:

```
kubectl -n work exec test-basdfj-bfasd -- sh -c 'env | grep TEST_ENV'
```

```
TEST_ENV=test-value
```

##### envFrom

 Если переменных среды окружения много, то рекомендуется использовать `ConfigMap`
 
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-configmap
data:
  TEST_ENV: test-value
  TEST_PORT: "80"
  TEST_PATH: /
```

```
kubectl -n work get cm
```

```
test-configmap
kube-root-ca.crt
```

`kube-root-ca.crt` добавляется автоматически при создании namespace.

Чтобы подключить env из `ConfigMap` нужно прописать в манифесте контейнера следующее:

```
envFrom:
  - configMapRef:
      name: test-configmap
```

При необходимости можно доставать не все переменные, а только необходимые:

```
env:
  - name: TEST_ENV
    key: test-value
  - name: TEST_PORT
    valueFrom:
      configMapKeyRef:
        name: test-configmap
        key: TEST_PORT
```


### Подключение ConfigMap как volume

Подключение `ConfigMap` как том в контейнер требует другой подход. В поде добавляют определение тома, в котором в качестве источника данных указывается `ConfigMap`. В контейнерах этот том монтируется уже в файловую систему контейнера.

```
spec:
  volumes:
    - name: test-volume
      configMap:
        name: test-configmap
  containers:
    - name: main
      image: busybox:1.28
      command: ["/bin/sh", "-c", "tail -f /dev/null"]
      volumeMounts:
        - name: test-volume
          mountPath: /etc/test
```

То есть сначала мы определяем все тома, что будут использоваться в поде через `volumes`, а потом уже в необходимых контейнерах подключаем тома `volumeMounts`.

Создадим `configMap` с файлами и `deployment` с томами и посмотрим что находится в директории `/etc/test`:

```
---
apiVerion: v1
kind: ConfigMap
metadata:
  name: test-configmap
data:
  file1.txt: |
    This is the content of file1.txt.
  file2.txt: |
    This is the content of file2.txt.
```

```
kubectl -n work exec test-dcjh9u-sdfljh -- sh -c 'ls -la /etc/test'
```

Мы видим несколько символьных ссылок, посмотрим содержимое реальной директории:

```
kubectl -n work exec test-dcjh9u-sdfljh -- sh -c 'ls -la /etc/test/..2026_09_03'
```

### Изменение содержимого ConfigMap

Попробуем изменить содержимое file1 в `ConfigMap`:

```
---
apiVerion: v1
kind: ConfigMap
metadata:
  name: test-configmap
data:
  file1.txt: |
    This is the new content of file1.txt.
  file2.txt: |
    This is the content of file2.txt.
```

Применим манифест `ConfigMap` и заметим, что данные внутри контейнера изменились.

Чтобы нельзя было изменять данные на лету, необходимо добавить `immutable: true` в манифест `ConfigMap`:

```
kind: ConfigMap
metadata:
  name: test-configmap
immutable: true
data:
  file1.txt: |
```


### Подключение одного файла

Для подключения только одного файла, без создания директории точки монтирования, будем использовать параметр `subPath`. В котором мы указываем файл в `ConfigMap`, а в `mountPath` пишем имя файла, на которое он будет отображен

```
volumeMounts:
  - name: test-volume
    mountPath: /etc/subfile.txt
    subPath: file1.txt
```

Причем в `mountPath` можно указать любое название для файла, но в `subPath` необходимо выбрать ключ, который в `ConfigMap` используется