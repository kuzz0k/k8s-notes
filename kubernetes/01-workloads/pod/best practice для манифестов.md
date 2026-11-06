##### Метки (labels)

Помогают управлять объектами и получать к ним доступ через селекторы, указываются в metadata. Желательно указывать как минимум название и версию.
```
metadata:
	name: my-pod
	namespace: work
	labels:
		app: nginx
		version: 1.27.1
```

##### Образ 

!Никогда не использовать `latest`! Важно всегда описывать конкретную версию, чтобы не было разногласий.
```
spec:
	containers:
		- name: nginx
		image: nginx:1.27.1
```

##### Ресурсы

```
spec:
	containers:
		- name: nginx
		image: nginx:latest
	ports:
		- containerPort: 80
	resources:
		requests:
			memory: "64Mi"
			cpu: "250m"
		limits:
			memory: "128Mi"
			cpu: "500m"
```
`requests` сразу резервирует ресурсы под контейнер, `limits` же ограничивают, если например будет больше памяти потребляться, то придет oomkilled и завершит

##### Пробы

Позволяют k8s проверять состояние контейнеров

Существует 4 механизма для реализации проверок:
- `exec` - выполнение командной строки внутри контейнера и возвращает код завершения (0 - успех, не 0 - неуспех)
- `tcpSocket` - отправка TCP-запроса на указанный порт
- `httpGet` - выполнение HTTP-запроса по указанному пути и порту, каждый раз nginx будет делать запись в логи, поэтому лучше использовать tcpSocket
- `grpc` - использование grpc для проверки состояние

Для проверки состояния можно использовать три типа проб:
- `Liveness Probe`— проверяет, живо ли приложение. При неудаче контейнер перезапускается.
- `Readiness Probe` — проверяет, готово ли приложение принимать трафик. Пока проверка не пройдена, Pod исключается из Service.
- `Startup Probe` — проверяет успешный запуск приложения. Пока она не завершится успешно, проверки Liveness и Readiness не выполняются. Используется для приложений с долгим стартом.

```
spec:
	containers:
		- name: nginx
		image: nginx:latest
	ports:
		- containerPort: 80
	resources:
		requests:
			memory: "64Mi"
			cpu: "250m"
		limits:
			memory: "128Mi"
			cpu: "500m"
	livenessProbe:
		tcpSocket:
			port: 80
		initialDelaySeconds: 15
		periodSeconds: 20
	readinessProbe:
		tcpSocket:
			port: 80
		initialDelaySeconds: 5
		periodSeconds: 10
```

