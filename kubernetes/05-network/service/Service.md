Доступ к приложению в k8s осуществляется через сервисы, которые представляют единую точку доступа к группе Pod'ов.

Поскольку Pod могут пересоздаваться и менять IP-адрес, Service обеспечивает стабильный IP и DNS-имя для доступа к приложению.

### ClusterIP

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: service-name
spec:
  selector:
    app: pod-selector
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

В приведённом выше примере создаётся сервис с именем service-name. Kubernetes присваивает сервису ip адрес и добавляет его данные в DNS сервер. После этого вы можете обратиться к сервису по его имени. В нашем случае сервис принимает запросы на 80-м порту и, если мы находимся в том же неймспейсе, мы можем обратиться к нему http://service-name. Если в другом, то с указанием неймспейса: http://service-name.namespace.svc.

Приходящие запросы сервис будет пересылать на порт 8080 подам с метками (label) app: selector. Если в системе будет несколько подов с таким селектором, сервис будет перераспределять запросы между ними. По умолчанию по алгоритму round robbin.

В качестве значения targetPort можно использовать имена портов. Конечно, если вы его описали при определении пода. Это удобно, если вы ссылаетесь на поды, у которых определены разные номера портов, но под одним именем.

**_Файл 01-_****_deployment_****_._****_yaml_****_._**

В deployment указано наличие двух реплик (подов) приложения. Объявляется порт приложения 8080, с названием tomcat. Приложению присваивается метка: app: tomcat.

**_Файл 02-_****_service_****_._****_yaml_****_._**

В разделе selector мы указываем метку приложения, на которые мы будем ссылаться. В разделе ports говорим, что к сервису нужно обращаться на 80-й порт. Запрос будет переслан приложению на порт, имеющий имя tomcat.

Если обратиться к любому поду напрямую, мы увидим ответ сервера tomcat.

```
curl 10.234.9.2:8080
<!doctype html>
Тут было много символов
<h3>Apache Tomcat/10.0.0-M10</h3></body></html>
```

Посмотрим подробнее на сервис.

```
kubectl get svc tomcat-main -o yaml
```

```yaml
apiVersion: v1
kind: Service
…
  name: tomcat-main
  namespace: default
…
spec:
  clusterIP: 10.233.47.171
  ports:
  - port: 80
    protocol: TCP
    targetPort: tomcat
  selector:
    app: tomcat
  sessionAffinity: None
  type: ClusterIP
```

Система выделила сервису **виртуальный** ip адрес:

```
clusterIP: 10.233.47.171
```

Адрес виртуальный. Это значит, что вы не найдете на машине Linux интерфейса с таким ip. Следует отметить, что за выдачу ip для сервисов отвечает kube-proxy, а не модуль IPAM драйвера сети Kubernetes.

Обратите внимание, что ip адреса сервисам выдаются из диапазона, определенного при помощи serviceSubnet, заданном при установке кластера.

По умолчанию создаются сервисы типа ClusterIP.

### Endpoint

Каким образом происходит связь между сервисом и подами? «За кадром» остался еще один элемент – endpoint.

```
kubectl get ep
NAME          ENDPOINTS                                                        AGE
kubernetes    192.168.218.171:6443,192.168.218.172:6443,192.168.218.173:6443   174m
tomcat-main   10.234.8.196:8080,10.234.9.2:8080                                3m17s
```

Посмотрим на него подробнее.

```
kubectl get ep tomcat-main -o yaml
```

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  …
  name: tomcat-main
  namespace: default
  …
subsets:
- addresses:
  - ip: 10.234.8.196
    nodeName: worker1.kryukov.local
    targetRef:
      kind: Pod
      name: tomcat-56db4566fd-xdctz
      namespace: default
     …
  - ip: 10.234.9.2
    nodeName: worker2.kryukov.local
    targetRef:
      kind: Pod
      name: tomcat-56db4566fd-8ffjg
      namespace: default
      …
  ports:
  - port: 8080
    protocol: TCP
```

В случае сервисов, использующих селектор, такие endpoints создаются автоматически. Система сама находит ip адреса подов, имеющих соответствующие метки, и формирует записи в endpoint.

### Сервисы без селекторов.

Сервисы без селекторов обычно используются для обращения за пределы кластера по ip адресу к какому либо приложению.

В качестве примера возьмем mail.ru

Определение сервиса:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mail-ru
spec:
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

**_Файл_** **_03-service-mail-ru.yaml_**

Порт 8080 у сервиса был поставлен в качестве эксперимента. Можно оставить его значение равным 80.

Определение endpoint. Имя сервиса и endpoint должны совпадать.

```yaml
---
apiVersion: v1
kind: Endpoints
metadata:
  name: mail-ru
subsets:
  - addresses:
      - ip: 217.69.139.200
      - ip: 94.100.180.200
      - ip: 217.69.139.202
      - ip: 94.100.180.201
    ports:
      - port: 80
```

**_Файл 04-endpoint-mail-ru.yaml_**

IP-адреса конечных точек не _должны быть:_ _loopback_ _(127.0.0.0/8 для_ _IPv__4, :: 1/128 для_ _IPv__6) или локальными для ссылки (169.254.0.0/16 и 224.0.0.0/24 для_ _IPv__4,_ _fe__80 :: / 64 для_ _IPv__6)._

Проверим, можем ли мы обращаться к новому сервису:

```
curl 10.233.99.119:8080
```

Работать с ip адресами не удобно. Запустим в кластере какой ни будь под и обратимся к сервисам по их именам.

```
kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools

curl mail-ru:8080
```

### EndpointSlice

При увеличении количества endpoints (читаем подов), ресурс приобретал просто огромные размеры.

При помощи слайса объект содержащий endpoint системы разбили на куски (слайсы). Теперь endpoints хранятся в EndpointSlice. Разбиение происходит автоматически. По умолчанию в одном slice хранится около 100 endpoints.

```
kubectl get endpointslice

kubectl get endpointslice tomcat-main-jf6dq -o yaml
```

EndpointSlice содержит набор портов, которые применяются ко всем endpoints. Мы можем в одном сервисе определить несколько портов. В результате на один сервис может быть создано несколько EndpointSlices.

### Headless Service

Что же такое headless сервис? Это просто запись А в системе DNS. Т.е. имя сервиса преобразуется не в виртуальный ip, его (ip) у нас нет, а сразу в ip пода.

Нужен для stateful приложений

Для того чтобы сделать headless сервис, достаточно указать `clusterIP: None` в сервисе:

```
spec:
	clusterIP: None
	selector:
```

### ExternalName

Применим манифест из файла 06-external-name.yaml.

Сервис типа ExternalName добавляет запись типа CNAME во внутренний DNS сервер Kubernetes.

Что у нас получается?

```
# kubectl get svc
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
mail-ru       ExternalName   <none>          mail.ru       <none>
```

Для этого сервиса не создаётся endpoint. Поэтому сразу переходим к запросам к DNS.

```
# kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
dnstools# dig mail-ru.default.svc.cluster.local
;; ANSWER SECTION:
mail-ru.default.svc.cluster.local. 5 IN CNAME   mail.ru.
mail.ru.                5       IN      A       217.69.139.202
mail.ru.                5       IN      A       94.100.180.200
mail.ru.                5       IN      A       94.100.180.201
mail.ru.                5       IN      A       217.69.139.200
dnstools# ping mail-ru
PING mail-ru (94.100.180.200): 56 data bytes
64 bytes from 94.100.180.200: seq=0 ttl=55 time=47.865 ms
64 bytes from 94.100.180.200: seq=1 ttl=55 time=47.892 ms
```

### NodePort

Сервисы типа NodePort открывают порт на каждой ноде кластера на сетевых интерфейсах хоста. Все запросы, приходящие на этот порт, будут пересылаться на endpoints, связанные с данным сервисом.

Диапазон портов, который можно использовать в NodePort — 30000-32767. Но его можно изменить при конфигурации кластера.

При описании сервиса необходимо явно указать его тип. Если не указать значение порта, при помощи параметра spec.ports.nodePort, порт присваивается автоматически из стандартного диапазона.

```
# kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP       PORT(S)
tomcat-nodeport   NodePort    10.233.43.49    <none>            80:31504/TC
```

Порт открывается на всех нодах кластера.