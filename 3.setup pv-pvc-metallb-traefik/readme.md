
#1. setup persistent volume (storageClass,pv,vpc)

## setup nfs server 
### server ip: 192.168.1.100
```
sudo apt install nfs-kernel-server
sudo mkdir -p /srv/nfs/kubedata
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
## by Helm chart

helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
   --set nfs.server=192.168.1.100 \
   --set nfs.path=/srv/nfs/kubedata 

kubectl get pods
kubectl get pv,pvc
kubectl get sc

k get sc nfs-client -o yaml 


```
kubectl create -f 1.rbac.yaml
kubectl create -f 2.class.yaml
kubectl create -f 3.deployment.yaml
kubectl create -f 4.test-claim.yaml
kubectl create -f 5.test-pod.yaml
/home/rajib/play/K8s-main/2.provision/1.vagrant/k8s-nfs/4.test-claim.yaml


kubectl get pods
kubectl get pv,pvc
kubectl get sc

kubectl get sc managed-nfs-storage -o yaml
```
# 2. Setup Metallb

```
kubectl create deploy nginx --image nginx
kubectl expose deploy nginx --port 80 --type=LoadBalancer
kubectl get svc nginx
```
### sipcalc install (Sipcalc is an ip subnet calculator for finding network range)

```
sudo apt-get install -y sipcalc
ip a s
    (vboxnet0:inet 172.16.16.1/24)
sipcalc
sipcalc 172.16.16.1/24
```
### Network range		- 172.16.16.0 - 172.16.16.255

## Installation By Manifest

```
kubectl -n kube-system get cm
kubectl -n kube-system describe cm kube-proxy | less
```

### Manifest file link chack commund
```
curl -s https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml | less
curl -s https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml | less
```
### To install MetalLB, apply the manifest:

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml

kubectl get ns
kubectl -n metallb-system get all
```
## Layer 2 Configuration

vim metallb.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.16.16.240-172.16.16.250
```
kubectl create -f metallb.yaml

## 2ed time test

```
kubectl create deploy nginx2 --image nginx
kubectl expose deploy nginx2 --port 80 --type=LoadBalancer
kubectl get svc nginx2

```
# 3. helm
https://github.com/helm/helm/releases
tar -zxvf helm-vxxx-xxxx-xxxx.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
helm version

helm version
helm repo list 
helm repo update
helm search repo traefik
helm show values traefik/traefik > /home/rajib/treafik-values.yaml
kubectl delete deploy traefik5 -n traefik
kubectl delete svc traefik5 -n traefik

# 4.Setup traefik 

```

helm version
helm repo list 
helm repo update
helm search repo traefik

helm show values traefik/traefik > /home/rajib/play/K8s-main/2.provision/1.vagrant/k8s-treafik/1.treafik-values.yaml
helm show values traefik/traefik > /home/rajib/treafik-values.yaml
helm install traefik traefik/traefik --values /home/rajib/treafik-values.yaml -n traefik --create-namespace
helm uninstall traefik -n traefik
helm install traefik traefik/traefik

kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
kubectl port-forward traefik3-667fc777ff-xp7g6 9000:9000

kubectl get pods --all-namespaces
kubectl get all --all-namespaces

kubectl get all -n traefik


````
























