# Migration plan:
## To be able to deploy our wordpress application to kubernetes, we would need to perform the following steps, in order:



* Stop the related wordpress docker-compose application on the docker server. 
* Open DNS zone file in a separate browser tab, and set `testblog.demo.wbitt.com` as CNAME for `traefik.demo.wbitt.com`. This will help propagate DNS changes, while we work on the actual migration.
* Perform a database dump of the existing MySQL database of this wordpress website from the old server.
* Copy the dump file from old db server to your local work computer.
* Create a database, user and password in the MySQL instance (running inside kubernetes cluster) for the wordpress application/site - through command line (using forwarded port).
* Load the database dump in the new database through mysql command line. 
* Make a tarball/zip/etc of the web content of your wordpress website from the old server, and copy it to your local work computer. 
* Create a PV and PVC for the wordpress deployment.
* Create the secrets for connecting the wordpress Deployment to the MySQL instance, and make sure that the wordpress deployment is configured to uses those secrets.
* Deploy the wordpress deployment, service and ingress. 
* Use `kubectl cp ...` command to copy the tarfile inside the wordpress container in `/tmp/`, and untar the files. Copy all the files from this location in `/tmp/<oldWPcontents>` to `/var/www/html/` over-writing everything. 
* Make sure you change ownership of all files to a user which the Apache web-server in that pod run as, i.e. UID 33, GID 33, using `chown -R 33:33 /var/www/html/` 
* Since you have overwritten the wordpress config file (`wp-config.php`) which was adjusted by the docker entrypoint, you will need to restart wordpress pod by simply killing it. 
* Since both the database and other web-content files are already there, this wordpress instance should start without any problem, showing the correct blog page and showing the media/pictures attached with this blog post.

**Note:** After the wordpress pod is started, you will be able to access it from the URL `testblog.demo.wbitt.com`. If you notice errors like: DB connection errors, empty pages, incorrectly rendered web pages, etc. This is because you have not yet migrated the database and not copied the web content directory of the existing website to the wordpress pod inside Kubernetes. 


**Note:** If the wordpress application was running as `https://` on the old server, then you will need to ensure that your new installation also runs on `https://` by correctly setup Traefik (with HTTPS) in the beginning. If you don't do this, and you run new installation on as plain HTTP, (or behind a plain HTTP reverse proxy), then your wordpress website will not render correctly. It happens because wordpress stores full URLs to various objects (such as pictures/images, etc) in the database, and tries to use those URLs as it is , when it needs to show those objects (pictures/images, etc). When the URLs mismatch, the picture file is not read from the file-system, and nothing is shown. This problem is very difficult to troubleshoot, because the wordpress pods's logs do not show this problem.

------ 

#### The best way to access your database instance in Kubernetes:

Setting up a web-UI in front of your database instance,(Adminer/phpMyAdmin/etc), with global access/reach-ability, is a horrible idea. Anyone with enough time and resources will continue to brute-force their way into the database server, through the web-UI.

The best/secure way is to port-forward database's service port to your local computer, using kubectl,  and then connecting to it through the localhost. This approach is very effective, but it expects that you have access to the kubernetes cluster, using kubectl. If that is not the case, then you do need some web interface to access your database instance. You may further secure it by setting up some firewall rules to allow access to the database instance only from selected IP addresses/ranges.

Another way could be to setup a **"jumpbox"** or **bastion host**, which has access to the cluster using kubectl, and forwards certain ports from the database service all the way to the jumpbbox. Then, setup SSH accounts on the jumpbox, for anyone wishing to connect to these database (forwarded) ports. These people will not actually connect directly to the database service on the jumpbox. Instead, they will connect to the jumpbox, and forward the related port to their local computer, and **then** use/connect-to the database service through that local port.

Another way could be to use an SSH server as a side-car inside the MySQL pod. This SSH server will allow only key-based access. The users can logon to this SSH server, and connect to the local mysql instance without a problem. Or, they can use this SSH server to forward MySQL port to their local work-computer, and use whatever MySQL client applications to talk to MySQL.

Above may seem a lot of work, but these are secure ways to access your database from outside the cluster.

------ 

