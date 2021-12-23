# Kubernetes setup:

To move our applications to Kubernetes, we would need to ensure that the individual needs of these applications are met. Certain things need to be in place. These are discussed next.

## The database service:
In current setup we have a single database server running three different database software, i.e. MySQL Postgres, MongoDB. Currently all applications connect to this database server on desired ports. We can have a similar setup in a slightly different way on Kubernetes. We can have three individual database services running as three separate "StatefulSet". This helps the database software to save it's state in a disk volume acquired using a "PersistentVolumeClaimTemplate". So MySQL , Postgres and MongoDB can have their individual StatefulSets. 

Lets talk about MySQL only. In Kubernetes terms, the MySQL instance needs:

* to be a StatefulSet object instead of Deployment. Read [this](https://github.com/KamranAzeem/kubernetes-katas/blob/master/08-storage-basic-dynamic-provisioning.md) to understand the "why".
* a publicly accessible docker container image. This will be `mysql:5.7` in our case.
* a disk volume to save it's data files. This will be a PVC of size 1 GB - for now.
* a secret (`MYSQL_ROOT_PASSWORD`), which will be used by the MySQL image to setup the MySQL instance correctly at first boot. 
* an internal/cluster service, so MySQL is accessible to all the services wishing to connect to it, within the same namespace.
* a way to be accessible / used by the admin from the internet, to be able to create databases and users for various applications / websites. For this, we would setup a very small and secure web interface for mysql, named `Adminer`. This Adminer software will have a "Deployment", a "Service" and an "Ingress", so we can access it from the internet. See note below.

**Note:** The database service will be setup by the main cluster administrator, so **this will be one time activity**. Though the process of creation of this service can be defined / saved as a github repository , in the form of `yaml` files, it does not need to be part of a CI/CD pipeline.

**Note:** Personally, I dislike the idea of providing global access to my database instance through any web interface. Refer to `The best way to access your database instance in Kubernetes` in this article.

## The ingress controller:
We will be setting up our website to be accessible over the internet. For that to work, we need an **ingress controller** in the cluster. This ingress controller will be a service - defined as `type: LoadBalancer`. In the docker-compose setup, we have Traefik running as the ingress controller - sort of. In our Kubernetes setup, we will continue to use Traefik (1.7) as Ingress controller. 

**Note:** The setup of Ingress Controller will also be a **one time activity** by the administrator.

### The individual applications:
Now we discuss the Kubernetes related needs of our applications. This is where developers will have the main interest, and they will have the main responsibility for deploying their applications on the cluster.

#### The WordPress application , and it's needs:
* It needs to be a **"Deployment"** , so we can scale up (and down) the number of replicas, depending on the load, which still able to serve the files on the (shared) disk from all the instances. This is only possible when you use a Deployment object, and not StatefulSet object.
* The Deployment will need an image. In this case, it uses a publicly available docker container image, so that is not a problem.
* The Deployment will need to know the location of MySQL database server, the DB for this wordpress installation , the DB username and passwords to connect to that database. This information cannot be part of the repository, so it is provided manually on the docker servers as `wordpress.env` file (as an example). On Kubernetes this information needs to be provided as environment variables. The question is, how? There are two ways. One, we create the secret manually from command line. The other way way is to setup the secrets as environment variables in the CI server. In a later article, we will be using CircleCI for our CI/CD needs. We will see both methods to get this done. 
* The Deployment will also need a persistent storage for storing various files this wordpress software will create. The same location will also hold any content uploaded by the user, for example pictures, etc. This will be a PVC, and will be created separately. It's definition of creation will not be part of the same file as the `deployment.yaml` . This is to prevent any accidents of old PVCs being deleted and new ones being created automatically resulting in data loss. This problem has been explain in another article of mine: [https://github.com/KamranAzeem/kubernetes-katas/blob/master/08-storage-basic-dynamic-provisioning.md](https://github.com/KamranAzeem/kubernetes-katas/blob/master/08-storage-basic-dynamic-provisioning.md) . Anyhow, the developer will be required to create the necessary PV/PVC once, and then make sure to **never delete the PVC**. Till the time the PVC is there, the data will be safe. 

**Note:** This example uses with wordpress, which is quite "stateful". What we want, from the developers is: applications as stateless as possible. i.e. There should be no involvement of saving any state, eliminating a need to acquire and maintain PVCs and PVs. **This is very important in application design.** 

#### The simple static/HTML/PHP application and it's needs:
* It needs to be a **"Deployment"** , so we can scale up (and down) the number of replicas, depending on the load, which still able to serve the files on the (shared) disk from all the instances. This is only possible when you use a Deployment object, and *not* StatefulSet object.
* The Deployment will need to use the docker image of our application. Kubernetes objects cannot build container images. (Recall: Kubernetes != Docker). So, for this to work, the image needs to be built outside/before the deployment process is carried out. If the image needs to be a private image, then GCP's [gcr.io](gcr.io) is ideal, as it can create private container images without requiring any extra steps at our end. Though you can choose any container registry of your choice.
* The Deployment will also need a (imaginary) configuration file mounted at `/config/site.conf` . One can argue that a configuration file (or files) can be baked into the image itself. In our case, the image will come with a default *site.conf*  file, and we can override it anytime by creating a config map with custom configuration, before creating the main deployment. This can be done manually, or through the CI server. In case of CI server, the entire configuration file will need to be stored an an environment variable in the CI server and then be used inside the deployment pipeline. I will show you that too.


So, the first thing at hand is to setup a Kubernetes cluster, and then setup MySQL database service and Traefik Ingress Controller inside it.


# Kubernetes setup plan:

* Open DNS zone file in a separate browser tab, and let it remain open. We will come back to it later.
* Deploy Traefik Ingress Controller, and create it's related service as `type:LoadBalancer` and obtain the public IP. Configure Traefik to use HTTPS using LetsEncrypt **Staging server**. You can use this guide: [https://github.com/KamranAzeem/kubernetes-katas/tree/master/ingress-traefik/https-letsencrypt-HTTP-Challenge](https://github.com/KamranAzeem/kubernetes-katas/tree/master/ingress-traefik/https-letsencrypt-HTTP-Challenge). It is best to keep the Traefik deployment and Traefik service definition files separate.
* Once you have the IP address for the load balancer, you create a DNS record `traefik.demo.wbitt.com` in the DNS zone file for `demo.wbitt.com`domain, and update that with the IP address assigned to the Traefik load-balancer service. 
* Now you setup ingress for `traefik-ui.demo.wbitt.com` , and see if Traefik can get staging certificate for it. If it does, (which it should), then it means LetsEncrypt is correctly setup.
* Reconfigure Traefik to use SSL certificates from LetsEncrypt's **Production servers**.
* Deploy MySQL as StatefulSet, and create related service. Do not setup MySQL as type LoadBalancer, NodePort; nor setup an ingress object against it. It must not be accessible directly from outside the cluster.

**Note:** It is VERY important that you set TTL for the DNS zone of the related domain to a low value, say "5 minutes". This will ensure that when you change DNS records, the change is propagated quickly across DNS servers around the world.

# Kubernetes setup:

I have setup a Kubernetes cluster on GCP/GKE. I have also ensured that I can access it using kubectl on my local work computer:

```
[kamran@kworkhorse mysql]$ kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
etcd-1               Healthy   {"health": "true"}   
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
[kamran@kworkhorse mysql]$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
gke-docker-to-k8s-default-pool-b0b5bbac-dx7z   Ready    <none>   6m25s   v1.14.10-gke.17
[kamran@kworkhorse mysql]$ 
```
