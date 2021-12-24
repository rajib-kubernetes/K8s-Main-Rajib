

## Now we setup the user student to use kubectl. This is part of the instructions in the kubeadmin init output above.

```
[root@kubeadm-node1 ~]# su - student
[student@kubeadm-node1 ~]$ mkdir -p $HOME/.kube
[student@kubeadm-node1 ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[student@kubeadm-node1 ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube -R
```

### Check if you can talk to the cluster/API server:
```
[student@kubeadm-node1 ~]$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
[student@kubeadm-node1 ~]$ kubectl get nodes
NAME            STATUS     ROLES    AGE   VERSION
kubeadm-node1   NotReady   master   12m   v1.12.2
```

# At least the cluster's components are ok!


