[![Kubernetes version](https://img.shields.io/badge/kubernetes-1.7.2-brightgreen.svg)](https://github.com/deimosfr/ansible-coreos-kubernetes)

Ansible Kubernetes Role for CoreOS Container Linux
==================================================

This role bootstrap a Kubernetes cluster based on CoreOS Container Linux for production usages.

Role features:
* Manage SSL certificats
* Configuration changes made easy (most of the config in the default file)
* Lot of checks during playbook run
* Manage Kubernetes upgrades
* Deploy Kubernetes DNS / Dashboard / Heapster...

[![asciicast](https://asciinema.org/a/97170.png)](https://asciinema.org/a/97170)

Requirements
------------

This role require Python to work because it's not present in CoreOS by default.

You can use those roles to install Python and make CoreOS installation (not mandatory but strongly recommanded):
* [deimosfr.coreos-ansible](https://github.com/deimosfr/ansible-coreos-ansible)
* [deimosfr.coreos-container-linux](https://github.com/deimosfr/ansible-coreos-container-linux)

You'll also need python libraries:
* dnspython

Role Variables
--------------

You can find [variable roles here](defaults/main.yml)

Example Playbook
----------------

First you need to fil an inventory. To make it simple, use a hosts file like this one (adapt IPs to your configuration):
```ini
[local]
localhost

[k8s-masters]
core01.nousmotards.com pub_ip=222.2.1.199 priv_ip=172.17.8.101
core02.nousmotards.com pub_ip=222.2.1.198 priv_ip=172.17.8.102
core03.nousmotards.com pub_ip=222.2.1.197 priv_ip=172.17.8.103

[k8s-workers]
core04.nousmotards.com pub_ip=222.2.1.196 priv_ip=172.17.8.104
core05.nousmotards.com pub_ip=222.2.1.195 priv_ip=172.17.8.105
core06.nousmotards.com pub_ip=222.2.1.194 priv_ip=172.17.8.106

[k8s-nodes:children]
k8s-masters
k8s-workers

[k8s-nodes:vars]
ansible_ssh_user=core
ansible_python_interpreter="/opt/python/bin/python"
```

Then you need to have a playbook to deploy Kubernetes like this one (adapt paths to your ansible directory):

```yaml
---

#########################
# ANSIBLE PREREQUISITES #
#########################

- name: coreos-ansible
  hosts: localhost
  gather_facts: False
  tasks:
    - include: ../../deimosfr.coreos-ansible/tasks/ansible_prerequisites.yml
  vars:
    ansible_python_interpreter: "/usr/bin/python"
    coreos_ansible_role_path: "{{playbook_dir}}/../../deimosfr.coreos-ansible"

- name: coreos-ansible
  hosts: k8s-nodes
  user: core
  become: yes
  gather_facts: False
  roles:
    - deimosfr.coreos-ansible
  vars:
    coreos_ansible_role_path: "{{playbook_dir}}/../../deimosfr.coreos-ansible"

#######################
# K8S SSL CERTIFICATS #
#######################

- name: kubernetes ssl
  hosts: localhost
  gather_facts: False
  tasks:
    - include: ../tasks/k8s_ssl_certs.yml
  vars_files:
    - ../defaults/main.yml

###############
# K8S MASTERS #
###############

- name: check kubernetes etcd prerequisites and deploy kubernetes masters configs
  hosts: k8s-masters
  user: core
  become: yes
  tasks:
    - include: ../tasks/k8s_etcd.yml
    - include: ../tasks/k8s_master_nodes.yml
  handlers:
    - include: ../handlers/main.yml
  vars_files:
    - ../defaults/main.yml
  vars:
    k8s_restart_kubelet: true

- name: configure kubernetes namespaces
  hosts: k8s-masters[0]
  user: core
  become: yes
  gather_facts: False
  tasks:
    - include: ../tasks/k8s_namespaces.yml
  vars_files:
    - ../defaults/main.yml

- name: validate pods are downloading or present
  hosts: k8s-masters
  user: core
  become: yes
  gather_facts: False
  tasks:
    - include: ../tasks/k8s_pods_checks.yml
  vars_files:
    - ../defaults/main.yml

###############
# K8S WORKERS #
###############

- name: deploy kubernetes workers configs
  hosts: k8s-workers
  user: core
  become: yes
  tasks:
    - include: ../tasks/k8s_workers_nodes.yml
  handlers:
    - include: ../handlers/main.yml
  vars_files:
    - ../defaults/main.yml
  vars:
    k8s_restart_kubelet: true

#############################
# Configure kubectl locally #
#############################

- name: configure kubectl locally
  hosts: k8s-nodes
  user: core
  become: yes
  gather_facts: False
  tasks:
    - include: ../tasks/k8s_kubectl_cfg.yml
  vars_files:
    - ../defaults/main.yml

######################
# K8S 3RD PARTY APPS #
######################

- name: deploy 3rd party apps
  hosts: k8s-masters[0]
  user: core
  gather_facts: False
  tasks:
    - include: ../tasks/k8s_dns_addon.yml
    - include: ../tasks/k8s_dashboard.yml
  vars_files:
    - ../defaults/main.yml

```

License
-------

GPLv3

Author Information
------------------

Pierre Mavro / deimosfr
