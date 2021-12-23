# Setup Traefik:

The first step is to setup Traefik with HTTPS enabled, using HTTP challenge. To achieve this, we use some extra files, i.e. `traefik.toml` and `dashboard-users.htpasswd`.

**Note:** Keep the deployment and service objects in separate files. [To do]

Remember to fix the email address in `traefik.toml` file.

```
[kamran@kworkhorse kubernetes]$ pwd
/home/kamran/Projects/Personal/github/docker-to-kubernetes/traefik/kubernetes

[kamran@kworkhorse kubernetes]$ kubectl  --namespace=kube-system  create configmap configmap-traefik-toml --from-file=traefik.toml
configmap/configmap-traefik-toml created

[kamran@kworkhorse kubernetes]$ htpasswd -c -b dashboard-users.htpasswd admin secretpassword

[kamran@kworkhorse kubernetes]$ kubectl  --namespace=kube-system  create secret generic secret-traefik-dashboard-users --from-file=dashboard-users.htpasswd
secret/secret-traefik-dashboard-users created

[kamran@kworkhorse kubernetes]$ kubectl apply -f traefik-rbac.yaml 
clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created


[kamran@kworkhorse kubernetes]$ kubectl apply -f traefik-deployment.yaml
serviceaccount/traefik-ingress-controller created
persistentvolumeclaim/pvc-traefik-acme-json created
deployment.extensions/traefik-ingress-controller created
service/traefik-ingress-service created
[kamran@kworkhorse kubernetes]$ 
```

Verify:
```
[kamran@kworkhorse kubernetes]$ kubectl --namespace=kube-system get pods
NAME                                                       READY   STATUS    RESTARTS   AGE
. . . 
traefik-ingress-controller-d76466dfc-zd59d                 1/1     Running   0          72s
```

```
[kamran@kworkhorse kubernetes]$ kubectl --namespace=kube-system get svc
NAME                      TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                                     AGE
default-http-backend      NodePort       10.0.15.173   <none>         80:31416/TCP                                21m
heapster                  ClusterIP      10.0.0.16     <none>         80/TCP                                      21m
kube-dns                  ClusterIP      10.0.0.10     <none>         53/UDP,53/TCP                               22m
metrics-server            ClusterIP      10.0.9.117    <none>         443/TCP                                     21m
traefik-ingress-service   LoadBalancer   10.0.14.84    35.228.250.6   80:30307/TCP,443:30999/TCP,8080:31041/TCP   111s
```



#### Deploy Traefik UI:

Find the IP of Traefik LB `35.228.250.6`, and adjust the DNS domain, so everything points to the traefik lb . Make sure that DNS has propagated:

```
[kamran@kworkhorse kubernetes]$ dig traefik-ui.demo.wbitt.com

;; QUESTION SECTION:
;traefik-ui.demo.wbitt.com.	IN	A

;; ANSWER SECTION:
traefik-ui.demo.wbitt.com. 299	IN	CNAME	traefik.demo.wbitt.com.
traefik.demo.wbitt.com.	299	IN	A	35.228.250.6

. . . 
[kamran@kworkhorse kubernetes]$ 
```

Create Traefik-web-UI deployment:

```
[kamran@kworkhorse kubernetes]$ kubectl apply -f traefik-webui-ingress.yaml 
service/traefik-web-ui created
ingress.extensions/traefik-web-ui created
[kamran@kworkhorse kubernetes]$ 

```

Verify by looking at Traefik web interface.


| ![images/traefik-web-ui-intial.png](images/traefik-web-ui-intial.png) |
| --------------------------------------------------------------------- |


**Note:** If you configured Traefik to obtain SSL certificates from **staging servers** , then at this point in time re-configure Traefik to use LetsEncrypt **production servers** . Perform the following steps:
* Delete the Traefik deployment
* Delete PVC used by Traefik
* Delete Traefik configmap (used for `traefik.toml`)
* Edit `traefik.toml` file and update address of certificate servers
* Re-create configmap for `traefik.toml`
* Re-create Traefik deployment and the related PVC by: `kubectl apply -f traefik-deployment.yaml`
