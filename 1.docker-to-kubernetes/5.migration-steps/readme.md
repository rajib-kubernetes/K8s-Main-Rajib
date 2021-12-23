# Actual migration / steps:

### Shutdown/stop the wordpress website on old server:

```
[kamran@kworkhorse ~]$ ssh root@web.witpass.co.uk 

[root@web testblog.demo.wbitt.com]# cd /home/containers-runtime/testblog.demo.wbitt.com/

[root@web testblog.demo.wbitt.com]# docker-compose -f docker-compose.server.yml down
Stopping testblogdemowbittcom_testblog.demo.wbitt.com_1 ... done
Removing testblogdemowbittcom_testblog.demo.wbitt.com_1 ... done
Network services-network is external, skipping
```

While you are in this server, make a tarball of the web-contents of this wordpress application.

```
[witpass@web ~]$ cd /home/containers-data/testblog.demo.wbitt.com/
[witpass@web testblog.demo.wbitt.com]$ tar czf /tmp/testblog.demo.wbitt.com.tar.gz .
```

Copy the tarball from your server to your local computer:
```
[kamran@kworkhorse ~]$ cd /tmp/
[kamran@kworkhorse tmp]$ scp witpass@web.witpass.co.uk:/tmp/testblog.demo.wbitt.com.tar.gz .
```

### Update DNS setting in DNS zone:
Update DNS setting in DNS zone file for the `demo.wbitt.com` zone, and verify with the following command:

```
[kamran@kworkhorse tmp]$ dig testblog.demo.wbitt.com

;; QUESTION SECTION:
;testblog.demo.wbitt.com.	IN	A

;; ANSWER SECTION:
testblog.demo.wbitt.com. 299	IN	CNAME	traefik.demo.wbitt.com.
traefik.demo.wbitt.com.	299	IN	A	35.228.250.6

[kamran@kworkhorse tmp]$ 
```


### Backup existing database:
Back up existing database related to this wordpress website, on the old DB server:


```
[root@db ~]# mysqldump db_testblog_demo_wbitt_com > db_testblog.demo.wbitt.com.dump

[root@db ~]# gzip -9 db_testblog.demo.wbitt.com.dump
```

Copy the dump-file from DB server to your local computer:
```
[kamran@kworkhorse ~]$ rsync root@db.witpass.co.uk:/root/db_testblog*.gz  /tmp/
```


### Restore the database to MySQL instance in Kubernetes:

Restoring the database using a web UI, such as `adminer` is straight forward. However, if you are not using a DB web UI, then the following steps need to be performed:

Forward the MySQL port from the Kubernetes cluster to your local computer, using kubectl. Do this in a separate shell/terminal. Before doing that, ensure that you are not running a local MySQL instance, because we need port 3306 on local computer to be available. 

```
[kamran@kworkhorse mysql]$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP    3h22m
mysql        ClusterIP   None         <none>        3306/TCP   11m
```

Run the following command in a separate terminal window and leave it running.
```
[kamran@kworkhorse mysql]$ kubectl port-forward svc/mysql 3306:3306 
Forwarding from 127.0.0.1:3306 -> 3306
Forwarding from [::1]:3306 -> 3306

(waits forever)
```

Open a new terminal. Make sure you are in the same directory where you copied the dump file from the old DB server. Unzip the dump file.

```
[kamran@kworkhorse tmp]$ pwd
/tmp

[kamran@kworkhorse tmp]$ ls -l
total 12504
-rw-r--r-- 1 kamran kamran   160570 Mar  7 19:20 db_testblog.demo.wbitt.com.dump.gz
-rw-r--r-- 1 kamran kamran 12637119 Feb 28 13:54 testblog.demo.wbitt.com.tar.gz

[kamran@kworkhorse tmp]$ gzip -d db_testblog.demo.wbitt.com.dump.gz 

[kamran@kworkhorse tmp]$ ls -l
total 12900
-rw-r--r-- 1 kamran kamran   569038 Mar  7 19:20 db_testblog.demo.wbitt.com.dump
-rw-r--r-- 1 kamran kamran 12637119 Feb 28 13:54 testblog.demo.wbitt.com.tar.gz
[kamran@kworkhorse tmp]$ 
```

Connect to this port (3306) on local computer, using `mysql` command, and create a MySQL database for your database, a user and a password for this user. 

```
[kamran@kworkhorse tmp]$ mysql -h 127.0.0.1 -u root -p
Enter password: 


MySQL [(none)]> create database testblog_demo_wbitt_com ;
Query OK, 1 row affected (0.061 sec)

MySQL [(none)]> grant all on testblog_demo_wbitt_com.* to 'testblog_demo_wbitt_com'@'%' identified by 'Pi+hd8cqfGc0oWOeZOCM8w==';
Query OK, 0 rows affected, 1 warning (0.025 sec)

MySQL [(none)]> flush privileges;
Query OK, 0 rows affected (0.028 sec)


MySQL [(none)]> select user,host,authentication_string from mysql.user;
+-------------------------+-----------+-------------------------------------------+
| user                    | host      | authentication_string                     |
+-------------------------+-----------+-------------------------------------------+
| root                    | localhost | *E5A99A335AAC5AAA1C688AD99906AF4C054C7283 |
| mysql.session           | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys               | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| root                    | %         | *E5A99A335AAC5AAA1C688AD99906AF4C054C7283 |
| testblog_demo_wbitt_com | %         | *681B6FC9E9359096E16EDED829F86C9D6C86E3BC |
+-------------------------+-----------+-------------------------------------------+
5 rows in set (0.098 sec)

MySQL [(none)]> 

```

Now exit the mysql session, and reconnect using these new credentials, to make sure that it works:

```
[kamran@kworkhorse tmp]$ mysql -h 127.0.0.1 -u testblog_demo_wbitt_com -D testblog_demo_wbitt_com -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [testblog_demo_wbitt_com]> show tables;
Empty set (0.028 sec)

MySQL [testblog_demo_wbitt_com]> exit
Bye
[kamran@kworkhorse tmp]$ 

```

Very good. Now load the DB dump you obtained from the old DB server, into this database:

```
[kamran@kworkhorse tmp]$ ls -l
total 12900
-rw-r--r-- 1 kamran kamran   569038 Mar  7 19:20 db_testblog.demo.wbitt.com.dump
-rw-r--r-- 1 kamran kamran 12637119 Feb 28 13:54 testblog.demo.wbitt.com.tar.gz

[kamran@kworkhorse tmp]$ mysql -h 127.0.0.1 -u testblog_demo_wbitt_com -D testblog_demo_wbitt_com -p < db_testblog.demo.wbitt.com.dump 
Enter password: 
[kamran@kworkhorse tmp]$ 
```

Reconnect and verify that the database has been restored successfully:

```
[kamran@kworkhorse tmp]$ mysql -h 127.0.0.1 -u testblog_demo_wbitt_com -D testblog_demo_wbitt_com -p
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [testblog_demo_wbitt_com]> show tables;
+-----------------------------------+
| Tables_in_testblog_demo_wbitt_com |
+-----------------------------------+
| wp_commentmeta                    |
| wp_comments                       |
| wp_links                          |
| wp_options                        |
| wp_postmeta                       |
| wp_posts                          |
| wp_term_relationships             |
| wp_term_taxonomy                  |
| wp_termmeta                       |
| wp_terms                          |
| wp_usermeta                       |
| wp_users                          |
+-----------------------------------+
12 rows in set (0.030 sec)

MySQL [testblog_demo_wbitt_com]> 
```

Database is restored on the new DB instance in Kubernetes. You can terminate the kubectl session running in the other terminal, being used to forward port 3306 to local computer.


### Setup WordPress Deployment:

Now we setup the Wordpress Deployment. Remember that the database related to this wordpress website is restored in Kubernetes, and we have the web content as a tarball on our local computer. 

First, we create the PVC required for this deployment.

```
[kamran@kworkhorse kubernetes]$ kubectl apply -f wordpress-pvc.yaml 
persistentvolumeclaim/pvc-testblog created

[kamran@kworkhorse kubernetes]$ kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-persistent-storage-mysql-0   Bound    pvc-b03e5db9-60b8-11ea-9327-42010aa600a1   1Gi        RWO            standard       50m
pvc-testblog                       Bound    pvc-c3b2d847-60bf-11ea-9327-42010aa600a1   1Gi        RWO            standard       18s
[kamran@kworkhorse kubernetes]$ 
```

Now, the PVC exists, but we cannot copy any files directly inside it. It has to be mounted inside a pod/container. We will use the wordpress deployment to mount this PVC at it's designated mount point, and copy the web-content inside it. For that we need the deployment; and the deployment needs secrets. So we have to perform the following steps.


#### Create secrets for WordPress Deployment:

```
[kamran@kworkhorse kubernetes]$ pwd
/home/kamran/Projects/Personal/github/docker-to-kubernetes/wordpress/kubernetes


[kamran@kworkhorse kubernetes]$ cat wordpress.env 
WORDPRESS_DB_HOST=mysql
WORDPRESS_DB_NAME=testblog_demo_wbitt_com
WORDPRESS_DB_USER=testblog_demo_wbitt_com
WORDPRESS_DB_PASSWORD=Pi+hd8cqfGc0oWOeZOCM8w==
[kamran@kworkhorse kubernetes]$ 
```

```
[kamran@kworkhorse kubernetes]$ ./create-wordpress-credentials.sh 
First, deleting the old secret: wordpress-credentials
Error from server (NotFound): secrets "wordpress-credentials" not found
Found wordpress.env file, creating kubernetes secret: wordpress-credentials
secret/wordpress-credentials created
[kamran@kworkhorse kubernetes]$ 
```

**Note:** At this point, ensure that DNS is already setup correctly, so `testblog.demo.wbitt.com` points to the IP address of the reverse proxy in the k8s cluster. 

```
[kamran@kworkhorse tmp]$ dig testblog.demo.wbitt.com

;; QUESTION SECTION:
;testblog.demo.wbitt.com.	IN	A

;; ANSWER SECTION:
testblog.demo.wbitt.com. 299	IN	CNAME	traefik.demo.wbitt.com.
traefik.demo.wbitt.com.	299	IN	A	35.228.250.6

[kamran@kworkhorse tmp]$ 
```

In the `wordpress-deployment.yaml` file, ensure that the ingress has correct host name set for the wordpress website. i.e `testblog.demo.wbitt.com` . Then create the deployment:

```
[kamran@kworkhorse kubernetes]$ kubectl apply -f wordpress-deployment.yaml 
deployment.apps/testblog created
service/testblog created
ingress.extensions/testblog created
[kamran@kworkhorse kubernetes]$ 
```

```
[kamran@kworkhorse kubernetes]$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
mysql-0                     1/1     Running   0          67m
testblog-54f855f697-n6bz2   1/1     Running   0          10s
[kamran@kworkhorse kubernetes]$ 
```

As soon as the deployment is created, this wordpress application will start. It will try to connect to the database, and then it will see that the document root (`/var/www/html`) in the pod/container is empty, (because it mounts the PVC we created just now), it will be populated by the fresh wordpress installation files. If you access `testblog.demo.wbitt.com` at this moment, you will (should) see your website - without any pictures/media, or plugins. Possibly showing some errors about missing plugins. In the screenshot below, I can't see the picture of cat I placed in my blog post when this site was running through docker-compose.

| ![images/website-without-pictures.png](images/website-without-pictures.png) |
| --------------------------------------------------------------------------- |


Don't worry. Wordpress does not have the file content yet. Just be sure that at this point, you **do not perform any actions on the wordpress web/admin interface**. Use `kubectl cp ...` command to copy the tarball of web content you obtained from the old server, inside this container, and unzip/untar it.

So lets move into the directory on local computer, which has the file content from previous server and copy the web-content tarball to this pod/container.

```
[kamran@kworkhorse tmp]$ pwd

[kamran@kworkhorse tmp]$ ls -l
total 12900
-rw-r--r-- 1 kamran kamran   569038 Mar  7 19:20 db_testblog.demo.wbitt.com.dump
-rw-r--r-- 1 kamran kamran 12637119 Feb 28 13:54 testblog.demo.wbitt.com.tar.gz

[kamran@kworkhorse tmp]$ kubectl cp testblog.demo.wbitt.com.tar.gz testblog-54f855f697-n6bz2:/tmp/ 
```

Now, login interactively to the pod/container on the OS level , using kubectl. Do some basic info collection steps:

```
root@testblog-54f855f697-n6bz2:/var/www/html# ls -ln
total 228
-rw-r--r--  1 33 33   420 Nov 30  2017 index.php
-rw-r--r--  1 33 33 19935 Jan  1  2019 license.txt
drwx------  2  0  0 16384 Mar  7 22:20 lost+found
-rw-r--r--  1 33 33  7368 Sep  2  2019 readme.html
-rw-r--r--  1 33 33  6939 Sep  3  2019 wp-activate.php
drwxr-xr-x  9 33 33  4096 Dec 18 22:16 wp-admin
-rw-r--r--  1 33 33   369 Nov 30  2017 wp-blog-header.php
-rw-r--r--  1 33 33  2283 Jan 21  2019 wp-comments-post.php
-rw-r--r--  1 33 33  2808 Mar  7 22:21 wp-config-sample.php
-rw-r--r--  1 33 33  3225 Mar  7 22:21 wp-config.php
drwxr-xr-x  5 33 33  4096 Mar  7 22:22 wp-content
-rw-r--r--  1 33 33  3955 Oct 10 22:52 wp-cron.php
drwxr-xr-x 20 33 33 12288 Dec 18 22:16 wp-includes
-rw-r--r--  1 33 33  2504 Sep  3  2019 wp-links-opml.php
-rw-r--r--  1 33 33  3326 Sep  3  2019 wp-load.php
-rw-r--r--  1 33 33 47597 Dec  9 13:30 wp-login.php
-rw-r--r--  1 33 33  8483 Sep  3  2019 wp-mail.php
-rw-r--r--  1 33 33 19120 Oct 15 15:37 wp-settings.php
-rw-r--r--  1 33 33 31112 Sep  3  2019 wp-signup.php
-rw-r--r--  1 33 33  4764 Nov 30  2017 wp-trackback.php
-rw-r--r--  1 33 33  3150 Jul  1  2019 xmlrpc.php
root@testblog-54f855f697-n6bz2:/var/www/html# 
```

Now, untar this tarball  inside /var/www/html/ .

```
root@testblog-54f855f697-n6bz2:/var/www/html# tar xzf /tmp/testblog.demo.wbitt.com.tar.gz 

root@testblog-54f855f697-n6bz2:/var/www/html# ls -l
total 228
-rw-r--r--  1 1001 1001   420 Nov 30  2017 index.php
-rw-r--r--  1 1001 1001 19935 Jan  1  2019 license.txt
drwx------  2 root root 16384 Mar  7 22:20 lost+found
-rw-r--r--  1 1001 1001  7368 Sep  2  2019 readme.html
-rw-r--r--  1 1001 1001  6939 Sep  3  2019 wp-activate.php
drwxr-xr-x  9 1001 1001  4096 Dec 18 22:16 wp-admin
-rw-r--r--  1 1001 1001   369 Nov 30  2017 wp-blog-header.php
-rw-r--r--  1 1001 1001  2283 Jan 21  2019 wp-comments-post.php
-rw-r--r--  1 1001 1001  2808 Feb 28 01:25 wp-config-sample.php
-rw-r--r--  1 1001 1001  3248 Feb 28 01:25 wp-config.php
drwxr-xr-x  5 1001 1001  4096 Feb 28 01:35 wp-content
-rw-r--r--  1 1001 1001  3955 Oct 10 22:52 wp-cron.php
drwxr-xr-x 20 1001 1001 12288 Dec 18 22:16 wp-includes
-rw-r--r--  1 1001 1001  2504 Sep  3  2019 wp-links-opml.php
-rw-r--r--  1 1001 1001  3326 Sep  3  2019 wp-load.php
-rw-r--r--  1 1001 1001 47597 Dec  9 13:30 wp-login.php
-rw-r--r--  1 1001 1001  8483 Sep  3  2019 wp-mail.php
-rw-r--r--  1 1001 1001 19120 Oct 15 15:37 wp-settings.php
-rw-r--r--  1 1001 1001 31112 Sep  3  2019 wp-signup.php
-rw-r--r--  1 1001 1001  4764 Nov 30  2017 wp-trackback.php
-rw-r--r--  1 1001 1001  3150 Jul  1  2019 xmlrpc.php
root@testblog-54f855f697-n6bz2:/var/www/html# 
```

Fix file ownership, which is **VERY IMPORTANT**:

```
root@testblog-54f855f697-n6bz2:/var/www/html# chown -R 33:33 .  

root@testblog-54f855f697-n6bz2:/var/www/html# ls -ln
total 228
-rw-r--r--  1 33 33   420 Nov 30  2017 index.php
-rw-r--r--  1 33 33 19935 Jan  1  2019 license.txt
drwx------  2 33 33 16384 Mar  7 22:20 lost+found
-rw-r--r--  1 33 33  7368 Sep  2  2019 readme.html
-rw-r--r--  1 33 33  6939 Sep  3  2019 wp-activate.php
drwxr-xr-x  9 33 33  4096 Dec 18 22:16 wp-admin
-rw-r--r--  1 33 33   369 Nov 30  2017 wp-blog-header.php
-rw-r--r--  1 33 33  2283 Jan 21  2019 wp-comments-post.php
-rw-r--r--  1 33 33  2808 Feb 28 01:25 wp-config-sample.php
-rw-r--r--  1 33 33  3248 Feb 28 01:25 wp-config.php
drwxr-xr-x  5 33 33  4096 Feb 28 01:35 wp-content
-rw-r--r--  1 33 33  3955 Oct 10 22:52 wp-cron.php
drwxr-xr-x 20 33 33 12288 Dec 18 22:16 wp-includes
-rw-r--r--  1 33 33  2504 Sep  3  2019 wp-links-opml.php
-rw-r--r--  1 33 33  3326 Sep  3  2019 wp-load.php
-rw-r--r--  1 33 33 47597 Dec  9 13:30 wp-login.php
-rw-r--r--  1 33 33  8483 Sep  3  2019 wp-mail.php
-rw-r--r--  1 33 33 19120 Oct 15 15:37 wp-settings.php
-rw-r--r--  1 33 33 31112 Sep  3  2019 wp-signup.php
-rw-r--r--  1 33 33  4764 Nov 30  2017 wp-trackback.php
-rw-r--r--  1 33 33  3150 Jul  1  2019 xmlrpc.php
root@testblog-54f855f697-n6bz2:/var/www/html# 
```

Check if images of my cat are there!
```
root@testblog-54f855f697-n6bz2:/var/www/html# ls -l wp-content/uploads/2020/02/767px-Cat_November_2010-1a*
-rw-r--r-- 1 www-data www-data   6239 Feb 28 01:35 wp-content/uploads/2020/02/767px-Cat_November_2010-1a-150x150.jpg
-rw-r--r-- 1 www-data www-data  15943 Feb 28 01:35 wp-content/uploads/2020/02/767px-Cat_November_2010-1a-225x300.jpg
-rw-r--r-- 1 www-data www-data 211437 Feb 28 01:35 wp-content/uploads/2020/02/767px-Cat_November_2010-1a.jpg
root@testblog-54f855f697-n6bz2:/var/www/html# exit
exit
[kamran@kworkhorse tmp]$ 
```

Very good. Now remember, when we over-wrote all the files, the wp-config.php was also over-written, with old information. Wordpress generates this file on pod/container startup, which means, we need to restart the pod, so wordpress can re-generate the config file with correct values. So, exit the pod/container, and simply kill it , so it can restart. 

```
[kamran@kworkhorse tmp]$ kubectl delete pod testblog-54f855f697-n6bz2
pod "testblog-54f855f697-n6bz2" deleted


[kamran@kworkhorse tmp]$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
mysql-0                     1/1     Running   0          82m
testblog-54f855f697-pzhd5   1/1     Running   0          10s
[kamran@kworkhorse tmp]$ 
```

Check logs of the new pod to see if there are any problems:

```
[kamran@kworkhorse tmp]$ kubectl logs -f testblog-54f855f697-pzhd5
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.32.0.16. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.32.0.16. Set the 'ServerName' directive globally to suppress this message
[Sat Mar 07 22:37:26.711030 2020] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.38 (Debian) PHP/7.3.15 configured -- resuming normal operations
[Sat Mar 07 22:37:26.711430 2020] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

I don't see any problems, so lets check the web page:

| ![images/website-with-cat-picture-after-migration.png](images/website-with-cat-picture-after-migration.png) |
| ----------------------------------------------------------------------------------------------------------- |

I see my cat! Hurray! It works!


Add another post, just to be sure! 

| ![images/blog-2.png](images/blog-2.png) |
| --------------------------------------- |

It works!


In [Part 3](Migration-Part-3.md), we deploy our simple HTML/PHP application using CI/CD.


# Additional Notes:
* To generate random passwords, I use: `openssl rand -base64 18`. I have set it up as an alias in my `~/.bashrc` as: `alias generate_random_16='openssl rand -base64 18'`
