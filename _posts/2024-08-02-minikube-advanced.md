---
layout: single
title: "Minikube Advance"
categories : Kubernetes
tag: [Kubernetes, Minikube, kubectl, Mount]
---

Minikube 제대로 활용하기

## Minikube 활용

### directory list

```bash
snoopy_kr@iMac Kubernetes % pwd
/Users/snoopy_kr/Working/VSCode/Docker/Kubernetes

snoopy_kr@iMac Kubernetes % ls -al
total 136
drwxrwxrwx  22 snoopy_kr  staff   704 Aug  2 17:29 .
drwxrwxrwx  16 snoopy_kr  staff   512 Apr 27 11:27 ..
-rwxrwxrwx   1 snoopy_kr  staff  6148 Aug  2 17:59 .DS_Store
drwxr-xr-x  14 snoopy_kr  staff   448 Aug  2 18:31 .git
-rw-r--r--   1 snoopy_kr  staff   549 Aug  2 17:23 .gitignore
drwxrwxrwx   4 snoopy_kr  staff   128 Nov 11  2022 mysql
-rwxrwxrwx   1 snoopy_kr  staff  1058 Aug  2 10:12 mysql.yml
drwxrwxrwx   4 snoopy_kr  staff   128 Nov 11  2022 tomcat
-rwxrwxrwx   1 snoopy_kr  staff  1045 Aug  2 10:12 tomcat.yml

snoopy_kr@iMac Kubernetes % cd tomcat

snoopy_kr@iMac tomcat % ls -al
total 0
drwxrwxrwx   4 snoopy_kr  staff  128 Nov 11  2022 .
drwxrwxrwx  22 snoopy_kr  staff  704 Aug  2 17:29 ..
drwxrwxrwx  13 snoopy_kr  staff  416 Nov 11  2022 conf
drwxrwxrwx  10 snoopy_kr  staff  320 Nov 11  2022 webapps

snoopy_kr@iMac Kubernetes % cd mysql

snoopy_kr@iMac mysql % ls -al
total 0
drwxrwxrwx   4 snoopy_kr  staff  128 Nov 11  2022 .
drwxrwxrwx  22 snoopy_kr  staff  704 Aug  2 17:29 ..
drwxrwxrwx   3 snoopy_kr  staff   96 Nov 11  2022 conf
drwxrwxrwx@ 21 snoopy_kr  staff  672 Aug  2 17:49 data
```

### Minikube 마운트

```bash
snoopy_kr@iMac ~ % minikube delete 
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /Users/snoopy_kr/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.

snoopy_kr@iMac ~ % minikube start --mount --mount-string="/Users/snoopy_kr/Working/VSCode/Docker/Kubernetes:/minikubeContainer/Kubernetes"
😄  minikube v1.33.1 on Darwin 13.6.7
✨  Automatically selected the docker driver
📌  Using Docker Desktop driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.44 ...
🔥  Creating docker container (CPUs=2, Memory=7792MB) ...
🐳  Preparing Kubernetes v1.30.0 on Docker 26.1.1 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```bash
snoopy_kr@iMac Kubernetes % kubectl apply -f mysql.yml 
deployment.apps/mysql created

snoopy_kr@iMac Kubernetes % kubectl apply -f tomcat.yml                
deployment.apps/tomcat created
service/tomcat created

snoopy_kr@iMac Kubernetes % kubectl get all                             
NAME                         READY   STATUS    RESTARTS   AGE
pod/mysql-f54bb9786-cbqbn    1/1     Running   0          13m
pod/tomcat-ff6bcc6b8-jd659   1/1     Running   0          12m
pod/tomcat-ff6bcc6b8-lklzv   1/1     Running   0          12m
pod/tomcat-ff6bcc6b8-mmp6z   1/1     Running   0          12m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          15m
service/mysql        NodePort    10.101.129.179   <none>        3306:30306/TCP   13m
service/tomcat       NodePort    10.110.201.249   <none>        8080:30080/TCP   12m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql    1/1     1            1           13m
deployment.apps/tomcat   3/3     3            3           12m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-f54bb9786    1         1         1       13m
replicaset.apps/tomcat-ff6bcc6b8   3         3         3       12m
```

### Minikube 터널링

```bash
snoopy_kr@iMac ~ % minikube service tomcat --url
http://127.0.0.1:55369
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

### Minikube 포트포워딩

```bash
snoopy_kr@iMac Kubernetes % kubectl port-forward svc/tomcat 8081:8080
Forwarding from 127.0.0.1:8081 -> 8080
Forwarding from [::1]:8081 -> 8080
Handling connection for 8081
Handling connection for 8081
```

### mysql.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  # replicas: 1
  selector:
    matchLabels:
      type: db
      service: mysql
  template:
    metadata:
      labels:
        type: db
        service: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7.27
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "**********"
        # ports:
        # - containerPort: 3306
        #   protocol: TCP
        volumeMounts:
        - name: datapath
          mountPath: /var/lib/mysql
        - name: confpath
          mountPath: /etc/mysql/conf.d
      volumes:
      - name : datapath
        hostPath:
          # path: /Users/snoopy_kr/Working/VSCode/Docker/Kubernetes:/minikubeContainer/Kubernetes/mysql/data
          path: /minikubeContainer/Kubernetes/mysql/data
          type: Directory
      - name : confpath
        hostPath:
          # path: /Users/snoopy_kr/Working/VSCode/Docker/Kubernetes:/minikubeContainer/Kubernetes/mysql/conf
          path: /minikubeContainer/Kubernetes/mysql/conf
          type: Directory

---

apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    protocol: TCP
    nodePort: 30306
  selector:
    type: db
    service: mysql
```
### tomcat.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat
spec:
  replicas: 3
  selector:
    matchLabels:
      type: app
      service: tomcat
  template:
    metadata:
      labels:
        type: app
        service: tomcat
    spec:
      containers:
      - name: tomcat
        # image: tomcat:8.5.45
        image: tomcat:9.0.24
        # ports:
        # - containerPort: 8080
        #   protocol: TCP
        volumeMounts:
        - name: confpath
          mountPath: /usr/local/tomcat/conf
        - name: webappspath
          mountPath: /usr/local/tomcat/webapps
      volumes:
      - name : confpath
        hostPath:
          # path: /Users/snoopy_kr/Working/VSCode/Docker/Kubernetes:/minikubeContainer/Kubernetes/tomcat/conf
          path: /minikubeContainer/Kubernetes/tomcat/conf
          type: Directory
      - name : webappspath
        hostPath:
          # path: /Users/snoopy_kr/Working/VSCode/Docker/Kubernetes:/minikubeContainer/Kubernetes/tomcat/webapps
          path: /minikubeContainer/Kubernetes/tomcat/webapps
          type: Directory

---

apiVersion: v1
kind: Service
metadata:
  name: tomcat
spec:
  type: NodePort
  ports:
  - port: 8080
    protocol: TCP
    nodePort: 30080
  selector:
    type: app
    service: tomcat
```