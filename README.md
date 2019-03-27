# Readme
Ansible template to create kubernetes cluster with the following specs:
* kubernetes 1.10.7
* docker v17.03.0-ce
* CNI container networking 0.5.2
* etcd 3.2.2
* calico v2.6
* RBAC
* Secure component between every components

# Dependency
You will need:
- linux/osx machine
- kubectl installed on that machine
- cfssl and cfssljson installed 

# Steps
### Kubernetes provision using ansible
Move to `ansible` directory and edit inventory files.
```
cd ../ansible
cp inventories/hosts.example inventories/hosts
vim inventories/hosts
```
Your inventory file will looks something like bellow. Match your nodes accordingly.
```
[etcd]
52.199.98.128
13.230.168.97
13.113.151.65

[controller]
18.179.174.13
18.179.196.179
52.68.64.239

[worker]
13.230.229.1
13.114.103.20
13.231.169.28

[kube:children]
controller
worker
```

Configure `group_vars/kube` file depends on your needs, at the very least, you have to change the `apiEndpoint` and `apiEndpointPublic` variable.
```
vim group_vars/kube
```

Generate ca for etcd and kubernetes:
```
cd files
mkdir etcd kubernetes
cfssl gencert -initca ca-csr.json | cfssljson -bare etcd/etcd-ca
cfssl gencert -initca ca-csr.json | cfssljson -bare kubernetes/kubernetes-ca
```

Run ansible:
```
ansible-playbook setup-all.yml
```

Test etcd:
```
ETCDCTL_API=3 etcdctl member list   --endpoints=https://127.0.0.1:2379   --cacert=/etc/etcd/ssl/etcd-ca.pem   --cert=/etc/etcd/ssl/etcd.pem   --key=/etc/etcd/ssl/etcd-key.pem
Provision kubernetes worker.
```

Save the token.

Kubeconfig file for admin will be generated at `files/kubernetes/ admin.kubeconfig`:

Run proxy:
```
kubectl --kubeconfig=files/kubernetes/admin.kubeconfig proxy
```

### Accessing dashboard
Kubernetes dashboard should be created automaticaly after all the ansible steps is done. To access the dashboard, use `kubectl proxy`.
```
kubectl --kubeconfig=files/kubernetes/admin.kubeconfig proxy
```
Now access Dashboard at:
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
