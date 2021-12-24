```

Now we setup the user student to use kubectl. This is part of the instructions in the kubeadmin init output above.

[root@kubeadm-node1 ~]# su - student

[student@kubeadm-node1 ~]$ mkdir -p $HOME/.kube

[student@kubeadm-node1 ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for student: 

[student@kubeadm-node1 ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube -R
Check if you can talk to the cluster/API server:

[student@kubeadm-node1 ~]$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
[student@kubeadm-node1 ~]$ 


[student@kubeadm-node1 ~]$ kubectl get nodes
NAME            STATUS     ROLES    AGE   VERSION
kubeadm-node1   NotReady   master   12m   v1.12.2
[student@kubeadm-node1 ~]$ 
At least the cluster's components are ok!


```
