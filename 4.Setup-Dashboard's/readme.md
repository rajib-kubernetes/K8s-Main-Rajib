### Rancher-
### DashboardUI- 
### Pometheus&Grafana- 
### EFK stack(lasticsearch, FluentBit & Kibana)-
### WeaveScope-

```
kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')" --dry-run
kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"

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


```
### K9sTerminalDashboard-
### Kontena Lens 
### Kail - Easy tool to view logs 





























