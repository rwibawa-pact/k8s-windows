# k8s-windows
docker & kubernetes for desktop

## 1. Install Docker Desktop for Windows
* [download from DockerHub](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)

## 2. Install `git`
* [download from git-scm.com](https://git-scm.com/downloads)

Then open the 'bash shell for git'

## 2. MySQL DB Server.
### 2.1 Create secret mysql-pass
```bash
$ kubectl create secret generic mysql-pass --from-literal=password=Ch@ng3M3!
secret/mysql-pass created

$ mkdir secret
$ kubectl get secret mysql-pass -o yaml > secret/mysql-pass.yaml
```

### 2.2 Create a persistent volume claim
For AWS kubernetes:
```bash
$ kubectl create -f mysql/aws/ebs-storage.yaml
storageclass.storage.k8s.io/aws-ebs-gp2 created
$ kubectl create -f mysql/aws/mysql-pvc.yaml
persistentvolumeclaim/mysql-pv-claim created
```

For windows kubernetes:
```bash
$ kubectl create -f mysql/windows/mysql-pvc.yaml 
persistentvolumeclaim/mysql-pv-claim created
```

### 2.3 Deploy MySQL Server.
[to set MySQL lower_case_table_names = 1](https://stackoverflow.com/questions/39942571/mysql-lower-case-table-names-1-with-kubernetes-yml-file-mysql-server-start-up).

```bash
$ kubectl create -f mysql/mysql-config.yaml 
configmap "mysql-config" created

$ kubectl create -f mysql/mysql-deployment.yaml
service/hh-mysql created
deployment.apps/hh-mysql created

$ kubectl get all
NAME                          READY     STATUS    RESTARTS   AGE
pod/hh-mysql-6dcc7679-nrq88   1/1       Running   0          19s

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/hh-mysql   ClusterIP   None         <none>        3306/TCP   19s

NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hh-mysql   1         1         1            1           19s

NAME                                DESIRED   CURRENT   READY     AGE
replicaset.apps/hh-mysql-6dcc7679   1         1         1         19s


$ winpty kubectl.exe exec -ti pod/hh-mysql-6dcc7679-nrq88 bash
root@hh-mysql-6dcc7679-nrq88:/# mysql -h localhost -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.43 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;
ERROR 1046 (3D000): No database selected
mysql> show databases;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| #mysql50#lost+found |
| mysql               |
| performance_schema  |
+---------------------+
4 rows in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'lower%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 1     |
+------------------------+-------+
2 rows in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'skip%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| skip_external_locking | ON    |
| skip_name_resolve     | ON    |
| skip_networking       | OFF   |
| skip_show_database    | OFF   |
+-----------------------+-------+
4 rows in set (0.00 sec)

mysql> CREATE USER 'dev'@'%' IDENTIFIED BY 'hello#';
mysql> GRANT ALL PRIVILEGES ON * . * TO 'dev'@'%';
mysql> CREATE SCHEMA `pact` ;

mysql> \q


$ kubectl port-forward svc/hh-mysql 3308:3306 &
Forwarding from 127.0.0.1:3308 -> 3306
Forwarding from [::1]:3308 -> 3306
Handling connection for 3308

```