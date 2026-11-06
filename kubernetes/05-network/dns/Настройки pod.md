## Hostname и subdomain

По умолчанию под имеет только краткое имя. Если по каким-то причинам ему надо задать FQDN имя, его можно определить непосредственно в yaml файле.

```
apiVersion: v1
kind: Pod
metadata:
  name: dnstools
  namespace: default
spec:
  hostname: dns
  subdomain: tools
  containers:
    - name: dnstools
      image: infoblox/dnstools:latest
      command:
        - sleep
        - "36000"
      imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

```
kubectl exec dnstools -it -- sh
hostname
hostname -f # FQDN
```

В итоге у контейнера будет установлено имя:

```
dns.tools.default.svc.cluster.local
```

Преобразование имени в ip адрес будет добавлено в файл /etc/hosts контейнера.

В зону DNS это преобразование добавляться не будет!

### Файл hosts

Для добавления записей в файл /etc/hosts контейнеров пода можно использовать `hostAliases`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dnstools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dnstools
  template:
    metadata:
      labels:
        app: dnstools
    spec:
      hostname: dns
      subdomain: tools
      hostAliases:
        - ip: 8.8.8.8
          hostnames:
            - dns.google.local
            - dns8.google.local
        - ip: 8.8.4.4
          hostnames:
            - dns4.google.local
      containers:
        - name: dnstools
          image: infoblox/dnstools:latest
          command:
            - sleep
            - "36000"
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
```

### Pod – конфигурация resolver

При описании пода можно вместо стандартного файла /etc/resolvr.conf настроить свой собственный вариант.

```
dnsPolicy: "None"
```

Эта политика заставляет под игнорировать настройки DNS Kubernetes. При этом требуется настроить DNS с использованием параметра

```
dnsConfig:
  nameservers:
    - 8.8.8.8
    - 8.8.4.4
  searches:
     - kryukov.biz
   options:
     - name: ndots
       value: "2"
```