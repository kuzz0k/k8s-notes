### Вручную

Создание сертификатов

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=testnginx.local"
```

Далее нужно создать secret с этими сертификатами

```
kubectl create secret tls local-selfsigned-tls \
  --cert=tls.crt --key=tls.key \
  --namespace traefik
```
### cert-manager

Cert-manager поддерживает `kind: Gateway`. Аннотации 100% совместимы с аннотациями для Ingress. Но! Чтобы cert-manager начал создавать сертификаты, его контроллер необходимо запустить с параметром `--enable-gateway-api`.

```
  annotations:
    cert-manager.io/cluster-issuer: dev-ca-issuer
    cert-manager.io/common-name: sample.kryukov.local
    cert-manager.io/subject-organizations: "home dev lab"
    cert-manager.io/subject-countries: "RU"
    cert-manager.io/subject-localities: "Moscow"
    cert-manager.io/duration: "9125h"
    cert-manager.io/renew-before: "360h"
    cert-manager.io/usages: "server auth"
    cert-manager.io/private-key-algorithm: RSA
    cert-manager.io/private-key-encoding: "PKCS8"
    cert-manager.io/private-key-size: "4096"
    cert-manager.io/private-key-rotation-policy: Always
```