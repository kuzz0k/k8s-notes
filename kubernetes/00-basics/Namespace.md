```
kubectl get ns
```
Namespace используются для изоляции ресурсов в пределах одного кластера. По умолчанию создаются 4 основных:
```
default           Active   3d23h
kube-node-lease   Active   3d23h
kube-public       Active   3d23h
kube-system       Active   3d23h
```
*default* - создается по умолчанию если не указывать название, не использовать!
*kube-node-lease* - хранятся объекты Lease, связанные с нодами кластера, позволяет kubelet отправлять сигналы, чтобы control plane могла обнаружить сбои
*kube-public* - могут читать все пользователи даже не прошедшие аутентификацию
*kube-system* - в нем располагаются приложения control plane
```
kubectl -n kube-system get pods
```
Чтобы создать свой namespace используется:
```
kubectl create ns work
```
