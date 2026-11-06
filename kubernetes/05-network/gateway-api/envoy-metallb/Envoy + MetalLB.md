[Envoy Gateway](https://gateway.envoyproxy.io)

![](images/envoy-gateway.png)

В отличие от Traefik, где LoadBalancer сервис и прочее создавалось автоматически, Envoy использует отдельный контроллер, который смотрит, что мы написали в манифестах и создает поды Envoy Proxy, причем их количество определяется количеством kind Gateway, автоматически создается такое же количество сервисов (по-умолчанию LB)

Т.е. Envoy Proxy - диннамическая кофигурации proxy киндов. Их можно запускать отдельно для Gateway, GatewayClass

### Установка Envoy

[Installation](https://gateway.envoyproxy.io/docs/tasks/quickstart/#installation)

Через helm:

```
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.9.2 -n envoy-gateway-system --create-namespace
```

### Envoy Proxy

[Envoy Proxy](https://gateway.envoyproxy.io/docs/api/extension_types/#envoyproxy)

Для каждого `Gateway` будет создаваться отдельный экземпляр прокси envoy. Конфигурация прокси происходит при помощи `kind: EnvoyProxy`. Это особенность непосредственно envoy и никакого отношения к GatewayAPI не имеет.

Конфигурацию можно привязать к `GatewayClass`. Тогда она будет распространяться на все экземпляры envoy proxy, относящиеся к этому классу. Либо делать отдельную конфигурацию для каждого экземпляра `Gateway`. Т.е. для отдельного экземпляра envoy.

В манифесте есть дополнительное логирование, а также `externalTrafficPolicy`. `externalTrafficPolicy` бывает Local и Cluster. Отличие в том, что Local направляет трафик только к тем подам, которые находятся на этой же ноде, Cluster направляет на все поды кластера.
[#### KubernetesServiceSpec](https://gateway.envoyproxy.io/docs/api/extension_types/#kubernetesservicespec)

Запустим сначала Proxy

Далее GatewayClass

### Gateway

Что Gateway, что HTTPRoute будут похожи на Traefik

### Установка MetalLb

Она будет идентичной как в Traefik: [Traefik + MetalLB → Установка MetalLB](../traefik-metallb/Traefik%20+%20MetalLB.md#установка-metallb)
