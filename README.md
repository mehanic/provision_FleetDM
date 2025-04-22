
## start helm chart
```
PASSWORD=$(openssl rand -base64 32) echo $PASSWORD

sudo helm repo add fleet https://fleetdm.github.io/fleet/charts
sudo helm repo update
helm install fleet fleet   --repo https://fleetdm.github.io/fleet/charts   --values values.yaml   --set auth.rootPassword="n2P/js1Jucn6aq2+plVwghdyAs0D7x8Zd176eOlmOzo="

export MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace "default" fleet-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
```

## generate tls
```
kubectl describe secret mysql -n default
  # Generate a private key
openssl genpkey -algorithm RSA -out server.key
   # Generate a self-signed certificate
openssl req -new -x509 -key server.key -out server.cert -days 365
ls
pwd
kubectl create secret tls fleet   --cert=server.cert   --key=server.key   --namespace=default

kubectl get pods -l app=fleet -o=jsonpath='{.items[0].spec.containers[0].image}'
fleetdm/fleet:v4.66.0   

kubectl create secret tls fleet \
  --cert=server.cert \
  --key=server.key \
  --namespace=default

  # Generate a private key
openssl genpkey -algorithm RSA -out server.key

# Generate a self-signed certificate
openssl req -new -x509 -key server.key -out server.cert -days 365
kubectl get secret fleet-tls -n default -o yaml

kubectl create secret tls fleet-tls --cert=/path/to/server.cert --key=/path/to/server.key -n default

# create for mounts
ls -ld /mnt/data/redis
ls -ld /mnt/data/mysql


#check debug services
kubectl run -i --tty --rm debug --image=busybox --restart=Never -- sh
```

## Warning  FailedScheduling  85s   default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

```
kubectl get pvc -n default
kubectl get pv

kubectl get pvc -n default 
NAME                                STATUS   VOLUME                              CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-fleet-mysql-0                  Bound    redis-data-pv1                      10Gi       RWO                           <unset>                 9h
redis-data-fleet-redis-master-0     Bound    data-fleet-mysql-0                  10Gi       RWO                           <unset>                 9h
redis-data-fleet-redis-replicas-0   Bound    redis-data-pv                       10Gi       RWO                           <unset>                 9h
redis-data-fleet-redis-replicas-1   Bound    redis-data-fleet-redis-replicas-1   10Gi       RWO                           <unset>                 8h
redis-data-fleet-redis-replicas-2   Bound    redis-data-fleet-redis-replicas-2   10Gi       RWO                           <unset>                 7h49m




```
# kubectl logs fleet-7d4f56887d-2cgqm -n default --previous
 ts=2025-04-22T03:44:23.434582643Z mysql="could not connect to db: dial tcp 127.0.0.1:3306: connect: connection refused, sleeping 0s"
 ts=2025-04-22T03:44:23.435559877Z mysql="could not connect to db: dial tcp 127.0.0.1:3306: connect: connection refused, sleeping 1s"
 ts=2025-04-22T03:44:24.43611866Z mysql="could not connect to db: dial tcp 127.0.0.1:3306: connect: connection refused, sleeping 2s"
 ts=2025-04-22T03:44:26.436929006Z mysql="could not connect to db: dial tcp 127.0.0.1:3306: connect: connection refused, sleeping 3s"


# mysql
```
sudo chown -R root:root /mnt/data/redis
sudo chown -R root:root /mnt/data/mysql
sudo chmod -R 775 /mnt/data/redis
sudo chmod -R 775 /mnt/data/mysql

securityContext:
  fsGroup: 1001  # Set an appropriate group ID
```

apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-pv-new
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data/redis  # Ensure this path exists or adjust based on your setup
```
# mysql="could not connect to db: dial tcp 127.0.0.1:3306: connect: connection refused"

kubectl exec -it fleet-mysql-0 -- env | grep MYSQL_ROOT_PASSWORD
MYSQL_ROOT_PASSWORD=mEWI251Cs5

kubectl exec -it fleet-mysql-0 -- mysql -u root -pmEWI251Cs5




kubectl exec -it fleet-mysql-0 -- env | grep MYSQL_ROOT_PASSWORD
MYSQL_ROOT_PASSWORD=mEWI251Cs5

kubectl exec -it fleet-mysql-0 -- mysql -u root -pmEWI251Cs5

```

```

kubectl exec -it fleet-6cd6f8bdf8-br6wd -- printenv | grep FLEET_MYSQL_ADDRESS
kubectl exec -it fleet-6cd6f8bdf8-2wxvq -- nslookup fleet-mysql.default.svc.cluster.local

kubectl describe secret fleet-tls -n default
echo "bjJQL2pzMUp1Y242YXEyK3BsVndnaGR5QXMwRDd4OFpkMTc2ZU9sbU96bz0=" | base64 --decode

kubectl get secret fleet-tls -n default -o jsonpath='{.data.tls\.crt}' | base64 --decode
kubectl get secret fleet-tls -n default -o jsonpath='{.data.tls\.key}' | base64 --decode




# You can change the authentication plugin for the user that Fleet is using to connect to MySQL by running the following SQL query inside your MySQL instance:

ALTER USER 'your_mysql_user'@'%' IDENTIFIED WITH caching_sha2_password;

# You can create a new user with caching_sha2_password if needed:

CREATE USER 'new_user'@'%' IDENTIFIED WITH caching_sha2_password BY 'your_password';

# If you have already updated the MySQL user to use caching_sha2_password, ensure that the Fleet MySQL user has the correct permissions to connect to the database. You can grant permissions and change authentication plugins via MySQL CLI. For example, to ensure the user has the right permissions and authentication:

GRANT ALL PRIVILEGES ON your_database.* TO 'your_mysql_user'@'%';
FLUSH PRIVILEGES;
ALTER USER 'your_mysql_user'@'%' IDENTIFIED WITH caching_sha2_password BY 'your_password';

# Once logged into MySQL, run:

SELECT user, host, plugin FROM mysql.user;
This will show which authentication plugin is being used for each user. Ensure that the user Fleet is using is set to caching_sha2_password.

# If the user is still using mysql_native_password, you can update it with the following command:

ALTER USER 'your_mysql_user'@'%' IDENTIFIED WITH caching_sha2_password BY 'your_password';
FLUSH PRIVILEGES;


# This will show if there are any restrictions on the root user. For example, it might be restricted to localhost or another specific host.

SELECT user, host, plugin FROM mysql.user WHERE user='root';


# Once logged in, run the following SQL query to list all MySQL users, This will give you a list of all users and the plugins used for authentication (e.g., mysql_native_password or caching_sha2_password).

SELECT user, host, plugin FROM mysql.user;

# After accessing MySQL, reset the root password:
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
FLUSH PRIVILEGES;

# If your_user exists but is not able to log in, it's possible that the user’s permissions are restricted. Check the privileges of the user by running:

SHOW GRANTS FOR 'your_user'@'localhost';

# If you identify that your_user needs privileges, you can grant them. For example, to give your_user access to all databases:
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;

# If you see that the user is restricted to localhost, try running this to allow the user to connect from any host:

UPDATE mysql.user SET host='%' WHERE user='your_user';
FLUSH PRIVILEGES;


# If you prefer to use the caching_sha2_password plugin (recommended for newer versions of MySQL), you can update the root user to use it. Run the following SQL:

ALTER USER 'root'@'%' IDENTIFIED WITH caching_sha2_password BY 'mEWI251Cs5';
FLUSH PRIVILEGES;

# If you plan to use a different user (like your_user), you can create or update that user with proper privileges.

CREATE USER 'your_user'@'%' IDENTIFIED WITH caching_sha2_password BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

# you continue to encounter issues with specific users being restricted to localhost, you can update the host for that user to % (to allow connections from any host)

UPDATE mysql.user SET host='%' WHERE user='your_user';
FLUSH PRIVILEGES;

# show rules 
SHOW GRANTS FOR 'fleet'@'%';

# if not enough we can add to it 
GRANT ALL PRIVILEGES ON *.* TO 'fleet'@'%';
FLUSH PRIVILEGES;


```

## work with mysql deployment
```
kubectl exec -it fleet-mysql-0 -- mysql -u root -pmEWI251Cs5
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5500
Server version: 8.0.34 Source distribution

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SELECT user, host, authentication_string FROM mysql.user WHERE user = 'fleet';
Empty set (0.01 sec)

mysql> CREATE USER 'fleet'@'%' IDENTIFIED BY 'n2P/js1Jucn6aq2+plVwghdyAs0D7x8Zd176eOlmOzo=';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT ALL PRIVILEGES ON fleet.* TO 'fleet'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT user, host, authentication_string FROM mysql.user WHERE user = 'fleet';
+-------+------+-------------------------------------------+
| user  | host | authentication_string                     |
+-------+------+-------------------------------------------+
| fleet | %    | *59AD16D6CE0914111449886F494EB4DAC950AD13 |
+-------+------+-------------------------------------------+
1 row in set (0.00 sec)

mysql> CREATE DATABASE fleet;
Query OK, 1 row affected (0.02 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| fleet              |
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.02 sec)

mysql> mysql> SELECT user, host, authentication_string FROM mysql.user WHERE user = 'fleet';
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'mysql> SELECT user, host, authentication_string FROM mysql.user WHERE user = 'fl' at line 1
mysql> +-------+------+-------------------------------------------+
    -> | user  | host | authentication_string                     |
    -> +-------+------+-------------------------------------------+
    -> | fleet | %    | *59AD16D6CE0914111449886F494EB4DAC950AD13 |
    -> +-------+------+-------------------------------------------+
    -> 1 row in set (0.00 sec)
    -> 
    -> mysql> CREATE DATABASE fleet;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '+-------+------+-------------------------------------------+
| user  | host | au' at line 1
mysql> Query OK, 1 row affected (0.02 sec)
    -> 
    -> mysql> SHOW DATABASES;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'Query OK, 1 row affected (0.02 sec)

mysql> SHOW DATABASES' at line 1
mysql> +--------------------+
    -> | Database           |
    -> ^C
mysql> 
mysql> 
mysql> GRANT ALL PRIVILEGES ON fleet.* TO 'fleet'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> mysql> SHOW DATABASES;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'mysql> SHOW DATABASES' at line 1
mysql> 
mysql> CREATE DATABASE fleet;
ERROR 1007 (HY000): Can't create database 'fleet'; database exists
mysql> GRANT ALL PRIVILEGES ON fleet.* TO 'fleet'@'%';
Query OK, 0 rows affected (0.02 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> mysql> SHOW DATABASES;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'mysql> SHOW DATABASES' at line 1
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| fleet              |
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.01 sec)

mysql> exit
Bye

```

## gather information about fleet
```
kubectl logs -l app=fleet
################################################################################
# ERROR:
#   Your Fleet database is not initialized. Fleet cannot start up.
#
#   Run `/usr/bin/fleet prepare db` to initialize the database.
################################################################################
################################################################################
# ERROR:
#   Your Fleet database is not initialized. Fleet cannot start up.
#
#   Run `/usr/bin/fleet prepare db` to initialize the database.
################################################################################
ts=2025-04-22T08:53:18.612669249Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 0s"
ts=2025-04-22T08:53:18.614637066Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 1s"
ts=2025-04-22T08:53:19.616948755Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 2s"
ts=2025-04-22T08:53:21.619944217Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 3s"
ts=2025-04-22T08:53:24.623988227Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 4s"
ts=2025-04-22T08:53:28.626418042Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 5s"
ts=2025-04-22T08:53:33.631212726Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 6s"
ts=2025-04-22T08:53:39.63340985Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 7s"
ts=2025-04-22T08:53:46.641078087Z mysql="could not connect to db: Error 1049 (42000): Unknown database 'fleet', sleeping 8s"
################################################################################
# ERROR:
#   Your Fleet database is not initialized. Fleet cannot start up.
#
#   Run `/usr/bin/fleet prepare db` to initialize the database.
################################################################################

# this job for execute ["/bin/sh", "-c", "/usr/bin/fleet prepare db"]
kubectl apply -f job.yaml 

kubectl get secrets -n default 
NAME                                   TYPE                 DATA   AGE
fleet                                  kubernetes.io/tls    2      8h
fleet-mysql                            Opaque               2      7h50m
fleet-redis                            Opaque               1      7h50m
fleet-tls                              kubernetes.io/tls    2      7h54m
my-app-secret                          Opaque               1      35d
mysql                                  Opaque               1      8h



```

## trouble with redis and how to decide 
kubectl logs -l app=fleet
Failed to start: initialize Redis: refresh cluster: redisc: all nodes failed
NOAUTH Authentication required.
Failed to start: initialize Redis: refresh cluster: redisc: all nodes failed
dial tcp 127.0.0.1:6379: connect: connection refused
Failed to start: initialize Redis: refresh cluster: redisc: all nodes failed
NOAUTH Authentication required.
Failed to start: initialize Redis: refresh cluster: redisc: all nodes failed
dial tcp 127.0.0.1:6379: connect: connection refused

```
            - name: REDIS_URL
              value: "fleet-redis-master.default.svc.cluster.local:6379"  # Redis service address
            - name: FLEET_MYSQL_ADDRESS
              value: "fleet-mysql.default.svc.cluster.local:3306"

            - name: FLEET_REDIS_PASSWORD
              valueFrom:
              secretKeyRef:
              name: fleet-redis  # Correct secret name
              key: redis-password  # Correct key for the Redis password


kubectl get secret fleet-redis -n default -o jsonpath='{.data.redis-password}' | base64 --decode
vWHBKw988p


kubectl get secret fleet-redis -n default -o json | jq '.data'
{
  "redis-password": "dldIQkt3OTg4cA=="
}



kubectl set env deployment/fleet FLEET_REDIS_ADDRESS=fleet-redis-master.default.svc.cluster.local:6379
deployment.apps/fleet env updated

kubectl set env deployment/fleet FLEET_REDIS_PASSWORD=$(kubectl get secret fleet-redis -n default -o jsonpath='{.data.redis-password}' | base64 --decode) -n default

kubectl get secret fleet-redis -n default -o jsonpath='{.data.redis-password}' | base64 --decode
kubectl set env deployment/fleet FLEET_REDIS_PASSWORD=$(kubectl get secret fleet-redis -n default -o jsonpath='{.data.redis-password}' | base64 --decode) -n default
kubectl get pods -l app=fleet -n default
kubectl logs -l app=fleet -n default
```

## trouble with mysql
```
env:        
            - name: FLEET_MYSQL_ADDRESS
              value: fleet-mysql.default.svc.cluster.local:3306
            - name: FLEET_REDIS_ADDRESS
              value: fleet-redis-master.default.svc.cluster.local:6379
```
            
## trouble with tls path

```
volumes:
  - emptyDir: {}
    name: tmp
  - name: fleet-tls
    secret:
      defaultMode: 420
      secretName: fleet-tls  # Update the secret name here
  - emptyDir:
      sizeLimit: 20Gi
    name: osquery-logs


volumeMounts:
  - mountPath: /tmp
    name: tmp
  - mountPath: /secrets/tls
    name: fleet-tls
    readOnly: true
  - mountPath: /logs
    name: osquery-logs

```
# check secrets that we can see in mount
```
kubectl exec -it check-tls-secrets -n default -- /bin/sh
/ # ls /secrets/tls
tls.crt  tls.key
/ # 
```


sometimes should check if we have terminated="open /secrets/tls/server.cert: no such file or directory"

```
volumes:
  - emptyDir: {}
    name: tmp
  - name: fleet-tls
    secret:
      defaultMode: 420
      secretName: fleet-tls  # Update the secret name here
  - emptyDir:
      sizeLimit: 20Gi
    name: osquery-logs



volumeMounts:
  - mountPath: /tmp
    name: tmp
  - mountPath: /secrets/tls
    name: fleet-tls
    readOnly: true  # Ensures the secret is mounted as read-only
  - mountPath: /logs
    name: osquery-logs

```
---------------
## should check that exist path to tls

```
- name: FLEET_SERVER_CERT
  value: /secrets/tls/server.cert
- name: FLEET_SERVER_KEY
  value: /secrets/tls/server.key


- name: FLEET_SERVER_CERT
  value: /secrets/tls/tls.crt
- name: FLEET_SERVER_KEY
  value: /secrets/tls/tls.key

```
```
kubectl logs -l app=fleet -n default

kubectl rollout restart deployment/fleet -n default

kubectl delete pod -l release=fleet
```
![fleet services ](1.png)



# metalb + traefik
```
└> $ sudo helm install traefik traefik/traefik \
  --namespace traefik --create-namespace \
  --set service.type=LoadBalancer
[sudo] password for mehanic: 
NAME: traefik
LAST DEPLOYED: Mon Apr 21 19:25:40 2025
NAMESPACE: traefik
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
traefik with docker.io/traefik:v3.3.6 has been deployed successfully on traefik namespace !



 └> $ sudo kubectl get all -n traefik 
NAME                           READY   STATUS    RESTARTS   AGE
pod/traefik-74f5c97947-6zwgc   1/1     Running   0          28s

NAME              TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/traefik   LoadBalancer   10.0.255.217   <pending>     80:30091/TCP,443:30741/TCP   29s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/traefik   1/1     1            1           29s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/traefik-74f5c97947   1         1         1       29s


 └> $ sudo  kubectl get nodes master -o wide
NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master   Ready    control-plane   58d   v1.30.6   192.168.0.227   <none>        Ubuntu 24.04.2 LTS   6.11.0-21-generic   containerd://1.7.27


 └> $ kubectl describe -n traefik svc traefik 
Name:                     traefik
Namespace:                traefik
Labels:                   app.kubernetes.io/instance=traefik-traefik
                          app.kubernetes.io/managed-by=Helm
                          app.kubernetes.io/name=traefik
                          helm.sh/chart=traefik-35.0.1
Annotations:              meta.helm.sh/release-name: traefik
                          meta.helm.sh/release-namespace: traefik
Selector:                 app.kubernetes.io/instance=traefik-traefik,app.kubernetes.io/name=traefik
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.0.255.217
IPs:                      10.0.255.217
Port:                     web  80/TCP
TargetPort:               web/TCP
NodePort:                 web  30091/TCP
Endpoints:                10.0.0.231:8000
Port:                     websecure  443/TCP
TargetPort:               websecure/TCP
NodePort:                 websecure  30741/TCP
Endpoints:                10.0.0.231:8443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>


 └> $ sudo kubectl get nodes master -o wide
NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master   Ready    control-plane   58d   v1.30.6   192.168.0.227   <none>        Ubuntu 24.04.2 LTS   6.11.0-21-generic   containerd://1.7.27


 └> $ sudo  kubectl get nodes master -o wide
NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master   Ready    control-plane   58d   v1.30.6   192.168.0.227   <none>        Ubuntu 24.04.2 LTS   6.11.0-21-generic   containerd://1.7.27


 └> $ kubectl get pods -n metallb-system
NAME                       READY   STATUS    RESTARTS   AGE
controller-c76b688-7dszb   1/1     Running   0          8m55s
speaker-mvzsh              1/1     Running   0          8m55s


 └> $ kubectl get ipaddresspools -n metallb-syste
No resources found in metallb-syste namespace.


 └> $ kubectl get ipaddresspools -n metallb-system 
NAME      AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
my-pool   true          false             ["192.168.0.240-192.168.0.250"]


 └> $ kubectl describe svc traefik -n traefik
Name:                     traefik
Namespace:                traefik
Labels:                   app.kubernetes.io/instance=traefik-traefik
                          app.kubernetes.io/managed-by=Helm
                          app.kubernetes.io/name=traefik
                          helm.sh/chart=traefik-35.0.1
Annotations:              meta.helm.sh/release-name: traefik
                          meta.helm.sh/release-namespace: traefik
                          metallb.universe.tf/ip-allocated-from-pool: my-pool
Selector:                 app.kubernetes.io/instance=traefik-traefik,app.kubernetes.io/name=traefik
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.0.255.217
IPs:                      10.0.255.217
LoadBalancer Ingress:     192.168.0.240
Port:                     web  80/TCP
TargetPort:               web/TCP
NodePort:                 web  30091/TCP
Endpoints:                10.0.0.231:8000
Port:                     websecure  443/TCP
TargetPort:               websecure/TCP
NodePort:                 websecure  30741/TCP
Endpoints:                10.0.0.231:8443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason            Age                    From                Message
  ----     ------            ----                   ----                -------
  Warning  AllocationFailed  9m33s (x2 over 9m33s)  metallb-controller  Failed to allocate IP for "traefik/traefik": no available IPs
  Normal   IPAllocated       4m                     metallb-controller  Assigned IP ["192.168.0.240"]
  Normal   nodeAssigned      4m                     metallb-speaker     announcing from node "master" with protocol "layer2"


#The curl -I http://192.168.0.240 command is making an HTTP #request to the IP address 192.168.0.240 and retrieving the #HTTP headers. The -I option tells curl to only show the #response headers, not the body of the response.


 └> $ curl -I http://192.168.0.240
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Mon, 21 Apr 2025 17:42:00 GMT
Content-Length: 19


```