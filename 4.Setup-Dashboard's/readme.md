### Rancher-
### DashboardUI- 
### Pometheus&Grafana- 
### EFK stack(lasticsearch, FluentBit & Kibana)-
### WeaveScope-

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
### K9sTerminalDashboard-

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

### Kontena Lens 
### Kail - Easy tool to view logs 





























