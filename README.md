# k8s-practice

Учебный стенд и конспекты по Kubernetes (k3s).

- [`terraform/`](terraform) — создание виртуалок под кластер
- [`ansible/`](ansible) — установка k3s и NFS-сервера
- [`kubernetes/`](kubernetes) — конспекты и манифесты
(оглавление — в [kubernetes/README.md](kubernetes/README.md))
- [`ecosystem/`](ecosystem) — то, что ставится поверх кубера: Argo CD, Helm, Kustomize, мониторинг
(оглавление — в [ecosystem/README.md](ecosystem/README.md))
- [zero-to-cluster](https://github.com/kuzz0k/zero-to-cluster) — проект, где все это собрано вместе: приложение, Gitea, CI/CD, Argo CD, мониторинг

## Поднять стенд

```
terraform init
terraform apply
```

```
ansible all -i inventory/hosts.ini -m ping
ansible-playbook -i inventory/hosts.ini playbooks/install-k3s.yml
ansible-playbook -i inventory/hosts.ini playbooks/setup-nfs-server.yml
```
