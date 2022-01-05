
#1. setup persistent volume (storageClass,pv,vpc)

## setup nfs server 
### server ip: 192.168.1.100
```
sudo apt install nfs-kernel-server
sudo mkdir -p srv/nfs/kubedata
sudo chown -R nobody:nogroup /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata

sudo vim /etc/exports
    /srv/nfs/kubedata (rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
sudo exportfs -a
sudo exportfs -ar
sudo exportfs -v

sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

## nfs clinte setup for checking

```
sudo apt install nfs-common
mount -t nfs 192.168.1.100:/srv/nfs/kubedata /mnt
mount | grep kubedata
umount /mnt

```

# k8s deployment persistent volume setup to existing cluster

```
kubectl create -f 1.rbac.yaml
kubectl create -f 2.class.yaml
kubectl create -f 3.deployment.yaml
kubectl create -f 4.test-claim.yaml
kubectl create -f 5.test-pod.yaml

kubectl get pods
kubectl get pv,pvc
kubectl get sc

kubectl delete -f 2.class
    #edit last line

kubectl get sc managed-nfs-storage -o yaml

```










# 2.setup mtllb 


# 3.setup traefik 





