

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
[student@kubeadm-node1 ~]$ kubectl get nodes

```
#  scp commond 
```
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo scp root@172.16.16.100:/etc/kubernetes/admin.conf ~/.kube/config

mkdir -p $HOME/.kube
sudo ssh-keygen -f "/root/.ssh/known_hosts" -R "kmaster.example.com"
sudo scp root@kmaster.example.com:/etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info
kubectl version

curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

```

## Solution:

### Modify the following files on all master nodes:

```
$ sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
Clear the line (spec->containers->command) containing this phrase: - --port=0
$ sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
Clear the line (spec->containers->command) containing this phrase: - --port=0
$ sudo systemctl restart kubelet.service
systemctl restart kubelet.service

```




