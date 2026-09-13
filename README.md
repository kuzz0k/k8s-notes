```
terraform init
terraform apply
```

```
ansible all -i inventory/hosts.ini -m ping
andible-playbook -i inventory/hosts.ini playbook/install-k3s.yml
ansible-playbook -i inventory/hosts.ini playbook/setup-nfs-server.yml
```

