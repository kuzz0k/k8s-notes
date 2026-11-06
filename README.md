# k8s-practice

Учебный стенд и конспекты по Kubernetes (k3s).

- [`terraform/`](terraform) — создание виртуалок под кластер
- [`ansible/`](ansible) — установка k3s и NFS-сервера
- [`kubernetes/`](kubernetes) — конспекты и манифесты
(оглавление — в [kubernetes/README.md](kubernetes/README.md))

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
