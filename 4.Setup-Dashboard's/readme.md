
### 1.Deploying metrics server

kubectl top no
kubectl top pods

### not working yet
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm search repo metrics-server
helm show values metrics-server/metrics-server > /home/rajib/metrics-server.values yaml

kubectl create ns metrics-server
helm install  metrics-server metrics-server/metrics-server --namespace metrics-server --values /home/rajib/metrics-server.values





### 2.Rancher-
### 3.Dashboard-UI- 
### 4.Pometheus&Grafana- 


### 5.k8s lens  

https://k8slens.dev/index.html

Dolwonlode k8slens and use it 



### 6.WeaveScope-

https://www.weave.works/docs/scope/latest/installing/#orchestrators

```
kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')" --dry-run
kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl version --short

kubectl get ns
kubectl -n weave get all
kubectl -n weave get pod
kubectl -n weave get deploy
kubectl -n weave get replicaset
kubectl -n weave get svc
kubectl get nodes -o wide

kubectl -n weave edit svc weave-scope-app

### Change clusterIP to LodeBlancer

### create IngressRoute
sudo vim 2.WeaveScope-IngressRoute.ymal

or 

echo 'apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: weave
  namespace: weave
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`weave.example.com`)
      kind: Rule
      services:
        - name: weave-scope-app
          port: 80' > WeaveScope-IngressRoute.ymal

kubectl create -f 2.WeaveScope-IngressRoute.ymal

weave.example.com

kubectl -n weave get svc



```
### 7.K9sTerminalDashboard-

```
kubectl version --short

https://github.com/derailed/k9s/releases
https://github.com/derailed/k9s/releases/download/v0.25.18/k9s_Linux_x86_64.tar.gz

cd /usr/local/bin
sudo wget https://github.com/derailed/k9s/releases/download/v0.25.18/k9s_Linux_x86_64.tar.gz

ls
sudo tar zxf k9s_Linux_x86_64.tar.gz
ls

which k9s

ls -l /usr/local/bin/k9s

### if not executable

sudo chmod +x /usr/local/bin/k9s

### config file location
.k9s/config.yml

k9s


```


### 8.EFK stack(Elasticsearch, FluentBit & Kibana)-

### 9.Kail - Easy tool to view logs 





























