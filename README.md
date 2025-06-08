# vagrant-ansible-k8s

credits : Xavki  

Provisionner les VMs avec le Vagrantfile
```bash
vagrant up
```

Configurer les VMs avec Ansible
```bash
ansible-playbook -i inventory.yml -u vagrant playbook.yml
```