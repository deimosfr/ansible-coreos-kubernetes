# Kubernetes upgrade procedure

To upgrade Kubernetes, this should be simple as described in the procedure:
https://coreos.com/kubernetes/docs/latest/kubernetes-upgrade.html

Just run the Ansible playbook:

```
ansible-playbook -i hosts roles/deimosfr.coreos-kubernetes/playbooks/upgrade/upgrade.yml -D
```