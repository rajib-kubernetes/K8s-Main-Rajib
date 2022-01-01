
# Accessing your cluster from machines other than the master node:

So far, we are able to talk to our Kubernetes cluster from node1, as user student. It is because that is where we have configured our `.kube/config` which is used by `kubectl` commands. To be able to access the cluster from some other computer, such as your work computer, etc, you need to copy the administrator kubeconfig file from your master node to your computer like this:

```
mkdir ~/.kube
scp student@<master-node-ip>:/home/student/.kube/config  ~/.kube/kubeadm-cluster.conf
kubectl --kubeconfig ~/.kube/kubeadm-cluster.conf get nodes
```

It might be a hassle to include `--kubeconfig ~/.kube/kubeadm-cluster.conf` in every kubectl command. You can setup a shell alias, or you can simply save it as `~/.kube/config` on your work computer. **WARNING** However, it is possible that you already have `~/.kube/config` file, which might contain configuration to one or more existing kubernetes clusters, and you don't want to accidently lose access to those clusters by over-writing it with this `.kube/config` . To be able to retain access to those clusters, and still also be able to use this kubeadm cluster, all from single kubectl, without specifying `--kubeconfig` everytime, you can use the KUBECONFIG environment variable. 

Lets start at master node, where we have a working copy of our `.kube/config` file.
First, we need to fix the default config we got from kubeadm. It is not very helpful in identifying which cluster is it. e.g. look at the following information. It looks very vague:

```
[student@kubeadm-node1 ~]$ kubectl config current-context
kubernetes-admin@kubernetes

[student@kubeadm-node1 ~]$ kubectl config get-clusters
NAME
kubernetes
[student@kubeadm-node1 ~]$
```

With a little `sed` , I changed the `.kube/config` for the student user. 


```
[student@kubeadm-node1 tmp]$ sed -i 's/kubernetes-admin/kubeadm-admin/g' $HOME/.kube/config

[student@kubeadm-node1 tmp]$ sed -i 's/kubernetes/kubeadm-cluster/g' $HOME/.kube/config
```


It now looks like the following:

```
[student@kubeadm-node1 ~]$ kubectl config current-context
kubeadm-admin@kubeadm-cluster

[student@kubeadm-node1 ~]$ kubectl config get-clusters
NAME
kubeadm-cluster
[student@kubeadm-node1 ~]$ 
```

Verify that it works:

```
[student@kubeadm-node1 ~]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-64d9675947-nw4kl   1/1     Running   0          3m53s
[student@kubeadm-node1 ~]$ 
```



Now, I copy this file from the master node to my work computer, inside `~/.kube/` directory, saving it as `kubeadm-cluster.conf`. 

```
[kamran@kworkhorse ~]$ scp student@kubeadm-node1:/home/student/.kube/config .kube/kubeadm-cluster.conf
config                                                                                                100% 5455     5.4MB/s   00:00    
[kamran@kworkhorse ~]$ 
```

I verify once that it works:

```
[kamran@kworkhorse ~]$ kubectl --kubeconfig=$HOME/.kube/kubeadm-cluster.conf  get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-64d9675947-nw4kl   1/1       Running   0          11m
[kamran@kworkhorse ~]$
```

Notice that my default .kube/config lists two clusters, one of them is production cluster for a client. I don't want to lose this configuration. 

```
[kamran@kworkhorse ~]$ kubectl config get-contexts
CURRENT   NAME                    CLUSTER                 AUTHINFO   NAMESPACE
*         minikube                minikube                minikube   
          client-gce.k8s.local   client-gce.k8s.local   admin      
[kamran@kworkhorse ~]$ 
```

So I have verified that without using `--kubeconfig` with `kubectl`, I can access my previous clusters. By using a specific configuration with `--kubeconfig` , I can access my kubeadm cluster. Is it possible to merge these two configurations? No. Not directly, which is actually a safety feature. Check this article for more detail: [https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable) .

I will solve it by using the KUBECONFIG environment variable as described in the above link. Here is how:

```
[kamran@kworkhorse ~]$ export KUBECONFIG=$KUBECONFIG:$HOME/.kube/config:$HOME/.kube/kubeadm-cluster.conf

[kamran@kworkhorse ~]$ kubectl config get-clusters
NAME
minikube
client-gce.k8s.local
kubeadm-cluster
[kamran@kworkhorse ~]$ 
```

If I do a `get-contexts`  or `get-clusters` now, I get all three clusters! Now, I can simply switch context and use my kubeadm-cluster, without using the `--kubeconfig=...` with each of my `kubectl` command.

```
[kamran@kworkhorse ~]$ kubectl config use-context kubeadm-admin@kubeadm-cluster
Switched to context "kubeadm-admin@kubeadm-cluster".


[kamran@kworkhorse ~]$ kubectl config get-contexts
CURRENT   NAME                            CLUSTER                 AUTHINFO        NAMESPACE
*         kubeadm-admin@kubeadm-cluster   kubeadm-cluster         kubeadm-admin   
          minikube                        minikube                minikube        
          client-gce.k8s.local           client-gce.k8s.local   admin           
[kamran@kworkhorse ~]$
```


```
[kamran@kworkhorse ~]$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-64d9675947-nw4kl   1/1       Running   0          20m
[kamran@kworkhorse ~]$ 
```

Hurray! It works!


Now, I just need to setup this `KUBECONFIG` permanently in my `.bash_profile` file so I don't have to do this every time I login to my computer. Depending on your situation and the way you start your session on your work computer, you may want to add this to any/all of:
* `/etc/profile`  (login shell)
* `~/.bash_profile` (login shell)
* `~/.bash_login` (login shell)
* `~/.profile` (login shell)
* `/etc/bash.bashrc` (non-login shell)
* `~/.bashrc` (non-login shell)

```
[kamran@kworkhorse ~]$ vi .bash_profile 

. . . 

PATH=$PATH:$HOME/.local/bin:$HOME/bin

KUBECONFIG=$KUBECONFIG:$HOME/.kube/config:$HOME/.kube/kubeadm-cluster.conf

export PATH KUBECONFIG
```

## Running kubectl from windows clients 
The procedure is almost the same as above except that you need to setup the KUBECONFIG environment variable somewhere in windows settings. 

A kubectl connectivity test from windows command line would look something like this:
```
kubectl --kubeconfig=C:\Users\joe\.kube\config.kubeadm get pods
```



**Note:**
The `admin.conf` file (`/etc/kubernetes/admin.conf` on master node, copied as `/home/student/.kube/config`) gives the user superuser privileges over the cluster. This file should be used very carefully. For normal users, it’s recommended to generate an unique credential, to which you whitelist privileges. You can do this with the `kubeadm alpha phase kubeconfig user --client-name <client-name>` command. This command will print out a KubeConfig file to STDOUT which you should save to a file and distribute to your user. After that, whitelist privileges by using `kubectl create (cluster)rolebinding`.

