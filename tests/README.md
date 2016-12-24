Ansible CoreOS Kubernetes role
==============================

Prerequisites
-------------

You first need to get a CoreOS cluster setup and ready. You can check
deimosfr.coreos role for more information

Configure Kubernetes
--------------------

Edit the available options in the defaults settings (defaults/main.yml) with
your needs.

Update the hosts file with your needs and you're good to go.

Install Kubernetes
------------------

To install kubernetes, there is a playbook example that you can easily run this way:

```
ansible-playbook -i roles/deimosfr.coreos-kubernetes/tests/hosts roles/deimosfr.coreos-kubernetes/tests/playbook_coreos_kubernetes.yml -D -e k8s_role_path="{{playbook_dir}}/.." 
```
