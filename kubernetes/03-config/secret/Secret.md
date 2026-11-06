Это специальный объект для конфидециальных данных, **Но важно помнить:** по умолчанию он лишь кодирует данные в base64, а не шифрует их
Максимальный размер - 1Мб
Поддерживает несколько типов: `Opaque`, `kubernetes.io/tls`, `kubernetes.io/dockerconfigjson` и другие

### Создание Secret

##### 1. Из литералов

```
kubectl -n work create secret generic my-secret --from-literal=username=admin --from-literal=password=secretpassword
```

`generic` создает `Secret` типа `Opaque`

```
kubectl -n work get secrets
```

Но кодируется всё в base64 и раскодировать никакого труда не составляет, правда для этого уже необходимо получить доступ к ноде

```
kubectl -n work get secret my-secret -o jsonpath='{.data.username}' | base64 --decode
```

##### 2. Из файлов

```
echo -n 'admin' > ./username.txt
echo -n 'secretpassword' > ./password.txt
kubectl -n work create secret generic my-secret --from-file=./username.txt --from-file=./password.txt
```

##### 3. Из YAML-манифеста

Для создания `Secret` из манифеста можно использовать:
- `data` - для уже закодированных в base64 данных
- `stringData` - для незакодированных данных (удобнее)

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # echo -n 'admin' | base64
  username: YWRtaW4=  # base64 encoded value
  # echo -n 'password' | base64
  password: cGFzc3dvcmQ=  # base64 encoded value
```

Чтобы закодировать сообщение необходимо ввести в терминале команду из комментариев

```
apiVerion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
stringData:
  username: admin
  password: password
```

При желании можно использовать одновременно и `data`, и `stringData`

##### 4. Тип kubernetes.io/tls

Нужен для хранения сертификатов и приватных ключей, для создания такого Secret сначала нужно сгенерировать сертификат и ключ с помощью OpenSSL

Генерация приватного ключа:

```
openssl genrsa -out tls.key 2048
```

Генерация самоподписанного сертификата:

```
openssl req -new -x509 -key tls.key -out tls.crt -days 365 -subj "/CN=myapp.domain.local"
```

После генерации сертификата и ключа можно создать `Secret` типа `tls`:

```
kubectl -n work create secret tls my-tls-secret --cert=tls.crt --key=tls.key
```

##### 5. Тип kubernetes.io/dockerconfigjson

Используется для хранения учетных данных Docker registry. Его можно создать двумя способами:

1. Используя файл конфигурации Docker:

```
kubectl create secret generic my-registry-secret \
	--from-file=.dockerconfigjson=/path/to/.docker/config.json \
	--type=kubernetes.io/dockerconfigjson
```

2.  Используя команду `kubectl create secret docker-registy`:

```
kubectl -n work create secret docker-registry my-registry-secret \
	--docker-server=https://registry.example.com \
	--docker-username=user \
	--docker-password=password \
	--docker-email=email@example.com
```

### Безопасность Secret

Несмотря на то, что `Secret` обеспечивает базовую защиту данных, важно понимать его ограничения:

- Данные хранятся в base64, что не является надеждым шифрованием
- Рекомендуется включить шифрование данных в etcd на уровне кластера
- Ограничивайте доступ к Secret через RBAC
- Используйте Secret только в тех подах, где это действительно необходимо