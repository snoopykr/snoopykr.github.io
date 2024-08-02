---
layout: single
title: "Minikube Basic"
categories : Kubernetes
tag: [Kubernetes, Minikube, kubectl]
---

Minikube을 활용한 Kubernetes 기초

## Minikube

### Minikube 설치

```bash
snoopy_kr@iMac ~ % curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 93.3M  100 93.3M    0     0  24.5M      0  0:00:03  0:00:03 --:--:-- 24.5M

snoopy_kr@iMac ~ % sudo install minikube-darwin-amd64 /usr/local/bin/minikube
Password:
```

### cluster 실행

```bash
snoopy_kr@iMac ~ % minikube start
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

### cluster 정보 확인

```bash
snoopy_kr@iMac ~ % kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:50440
CoreDNS is running at https://127.0.0.1:50440/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### kubectl의 버전 확인 

```bash
snoopy_kr@iMac ~ % kubectl version
Client Version: v1.30.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.30.0
```

### 노드 정보 확인 확인 

```bash
snoopy_kr@iMac ~ % kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   6h40m   v1.30.0
```

### 기본 Pod 확인

```bash
snoopy_kr@iMac ~ % kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7db6d8ff4d-r96s6           1/1     Running   1 (12m ago)   6h50m
kube-system   etcd-minikube                      1/1     Running   1 (12m ago)   6h50m
kube-system   kube-apiserver-minikube            1/1     Running   1 (12m ago)   6h50m
kube-system   kube-controller-manager-minikube   1/1     Running   1 (12m ago)   6h50m
kube-system   kube-proxy-ln4xx                   1/1     Running   1 (12m ago)   6h50m
kube-system   kube-scheduler-minikube            1/1     Running   1 (12m ago)   6h50m
kube-system   storage-provisioner                1/1     Running   2 (11m ago)   6h50m
```

### cluster 일시정지/재가동

```bash
snoopy_kr@iMac ~ % minikube pause
⏸️  Pausing node minikube ... 
⏯️  Paused 14 containers in: kube-system, kubernetes-dashboard, storage-gluster, istio-operator

snoopy_kr@iMac ~ % minikube unpause
⏸️  Unpausing node minikube ... 
⏸️  Unpaused 14 containers in: kube-system, kubernetes-dashboard, storage-gluster, istio-operator
```

### cluster 종료

```bash
snoopy_kr@iMac ~ % minikube stop 
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
🛑  1 node stopped.
```

### cluster 삭제

```bash
snoopy_kr@iMac ~ % minikube delete 
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /Users/snoopy_kr/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

## Pod

### 한개의 container를 가진 Pod

```bash
snoopy_kr@iMac Basic % kubectl apply -f single-container.yaml
pod/single-container-pod created

snoopy_kr@iMac Basic % kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
single-container-pod   1/1     Running   0          14s

snoopy_kr@iMac Basic % kubectl logs single-container-pod
hello
hello
hello
hello
hello

snoopy_kr@iMac Basic % kubectl delete pods single-container-pod
pod "single-container-pod" deleted
```

### 여러개의 container를 가진 Pod

```bash
snoopy_kr@iMac Basic % kubectl apply -f multiple-containers.yaml 
pod/multiple-containers-pod created

snoopy_kr@iMac Basic % kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
multiple-containers-pod   2/2     Running   0          12s

snoopy_kr@iMac Basic % kubectl logs multiple-containers-pod
Defaulted container "ubuntu1" out of: ubuntu1, ubuntu2
hello1
hello1
hello1
hello1
hello1

snoopy_kr@iMac Basic % kubectl logs multiple-containers-pod -c ubuntu1
hello1
hello1
hello1
hello1
hello1

snoopy_kr@iMac Basic % kubectl logs multiple-containers-pod -c ubuntu2
hello2
hello2
hello2
hello2
hello2

snoopy_kr@iMac Basic % kubectl delete pods multiple-containers-pod
pod "multiple-containers-pod" deleted
```

### Volume 공유

```bash
snoopy_kr@iMac Basic % kubectl apply -f volume-share-pod.yaml 
pod/volume-share-pod created

snoopy_kr@iMac Basic % kubectl logs volume-share-pod -c ubuntu2 -f
hello1
hello1
hello1
hello1
hello1
^C

snoopy_kr@iMac Basic % kubectl delete pods volume-share-pod
pod "volume-share-pod" deleted
```

### Network 공유

```bash
snoopy_kr@iMac Basic % kubectl apply -f network-share-pod.yaml         
pod/network-share-pod created

snoopy_kr@iMac Basic % kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
network-share-pod   2/2     Running   0          45s

snoopy_kr@iMac Basic % kubectl logs network-share-pod -c ubuntu-curl -f

8080 container
8080 container
8080 container
8080 container
8080 container
^C

snoopy_kr@iMac Basic % kubectl delete pods network-share-pod
pod "network-share-pod" deleted
```

## Controller

### Controller 종류

- ReplicaSet - Pod의 집합을 관리한다. ReplicaSet의 선언시 Pod의 갯수, container image 등을 선언해 놓으면 계속 해당 조건을 만족시키도록 Pod를 관리해준다.

- Deployment - ReplicaSet의 상위 개념으로 ReplicaSet을 관리한다. 즉 ReplicaSet, Pod가 모두 관리된다는 뜻이며 몇가지 Pod update 방식도 제공해주기 때문에 더욱 편리하게 배포 관리가 가능하다.

- StatefulSet - 이름에서 유추할 수 있듯이 데이터베이스 등과 같이 상태가 유지되어야 하는 application을 지원하기 위한 controller 이다. Application이 종료 되더라도 연결되었던 저장소는 그대로 남아 있어 새로운 Application이 실행되면 기존의 저장소와 연결되어 상태를 유지시켜 준다.

- DaemonSet - Kubernetes cluster 내에 생성된 모든 Node에 대해 배포하고자 하는 Pod가 있다면 DasemonSet을 사용할 수 있다. Node가 생성되면 DaemonSet에 정의된 Pod를 생성하고 Node가 사라지면 생성된 Pod도 사라진다.

- Job - 계속 running 되는 Pod를 생성하는 것이 아닌, Pod를 생성하여 원하는 작업을 한 후 종료하고자 할 때 사용할 수 있는 controller 이다. 환경에 따라 다양하게 사용될 수 있으며 Pod 생성 전략, 작업 실패시 전략 등 다양하게 설정을 변경하여 사용할 수 있다.

- CronJob - Cron 표현식을 사용하여 원하는 시점에 바로 위에서 확인한 Job을 생성해 주는 controller 이다.

## ReplicaSet

### ReplicaSet 형식

- replicas - ReplicaSet이 유지하고자 하는 Pod의 갯수이다. ReplicaSet은 지속적으로 Pod의 갯수를 확인하여 정상적으로 동작하고 있는 Pod의 갯수를 replicas에 정의한 갯수와 동일하게 맞춰준다.

- selector - ReplicaSet이 관리하고자 하는 Pod를 선택하기 위한 조건을 정의하는 필드이다. ReplicaSet은 아무 Pod나 선택해서 관리하는 것이 아니라, 관리하고자 하는 Pod를 selector 필드를 통해 선택하여 관리하게 된다.

- template - ReplicaSet을 통해 생성되고 관리되고자 하는 Pod에 대해 정의하는 필드이다. Pod를 정의할때와 마찬가지로 metadata와 spec에 대해 정의해주면 된다. Pod의 metadata에 정의된 label은 반드시 ReplicaSet의 selector와 match가 되어야한다.

```bash
snoopy_kr@iMac Basic % kubectl apply -f replicaset.yaml
replicaset.apps/ubuntu created

snoopy_kr@iMac Basic % kubectl get replicasets
NAME     DESIRED   CURRENT   READY   AGE
ubuntu   3         3         3       8s

snoopy_kr@iMac Basic % kubectl get pods                     
NAME           READY   STATUS    RESTARTS   AGE
ubuntu-rd8lr   1/1     Running   0          105s
ubuntu-w76rk   1/1     Running   0          105s
ubuntu-zvhs8   1/1     Running   0          105s

snoopy_kr@iMac Basic % kubectl delete replicasets ubuntu    
replicaset.apps "ubuntu" deleted

snoopy_kr@iMac Basic % kubectl get pods                 
NAME           READY   STATUS        RESTARTS   AGE
ubuntu-rd8lr   1/1     Terminating   0          3m46s
ubuntu-w76rk   1/1     Terminating   0          3m46s
ubuntu-zvhs8   1/1     Terminating   0          3m46s

snoopy_kr@iMac Basic % kubectl get pods                                             
No resources found in default namespace.
```

```bash
snoopy_kr@iMac Basic % kubectl apply -f manual-pod.yaml 
pod/manual-pod created

snoopy_kr@iMac Basic % kubectl get pods                
NAME         READY   STATUS    RESTARTS   AGE
manual-pod   1/1     Running   0          5s

snoopy_kr@iMac Basic % kubectl apply -f replicaset.yaml 
replicaset.apps/ubuntu created

snoopy_kr@iMac Basic % kubectl get pods                
NAME           READY   STATUS              RESTARTS   AGE
manual-pod     1/1     Running             0          43s
ubuntu-27wgj   0/1     ContainerCreating   0          2s
ubuntu-5lds6   0/1     ContainerCreating   0          2s

snoopy_kr@iMac Basic % kubectl delete replicasets ubuntu    
replicaset.apps "ubuntu" deleted

snoopy_kr@iMac Basic % kubectl get pods                 
NAME           READY   STATUS        RESTARTS   AGE
manual-pod     1/1     Terminating   0          2m9s
ubuntu-27wgj   1/1     Terminating   0          88s
ubuntu-5lds6   1/1     Terminating   0          88s
```

```bash
snoopy_kr@iMac Basic % kubectl apply -f replicaset.yaml 
replicaset.apps/ubuntu created

snoopy_kr@iMac Basic % kubectl get pods                
NAME           READY   STATUS              RESTARTS   AGE
ubuntu-6b4vr   0/1     ContainerCreating   0          2s
ubuntu-j44pn   0/1     ContainerCreating   0          2s
ubuntu-jk9r9   0/1     ContainerCreating   0          2s

snoopy_kr@iMac Basic % kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
ubuntu-6b4vr   1/1     Running   0          6s
ubuntu-j44pn   1/1     Running   0          6s
ubuntu-jk9r9   1/1     Running   0          6s

snoopy_kr@iMac Basic % kubectl get pods ubuntu-jk9r9 -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-08-01T15:52:52Z"
  generateName: ubuntu-
  labels:
    app: ubuntu
    env: test
  name: ubuntu-jk9r9
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: ubuntu
    uid: 37fb86e1-b7be-4b92-9960-ad420f290217
  resourceVersion: "4976"
  uid: f49bde66-fa90-4dff-ba38-743bbd3840fe
spec:
  containers:
  - args:
    - -c
    - while true; do echo hello; sleep 1; done
    command:
    - /bin/sh
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-ctfr9
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-ctfr9
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-08-01T15:52:58Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-08-01T15:52:52Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-08-01T15:52:58Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-08-01T15:52:58Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-08-01T15:52:52Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://6ef7980956889ffefdc71de8bbd84e265e57227ba3858513a7334cefdb9ff91f
    image: ubuntu:latest
    imageID: docker-pullable://ubuntu@sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
    lastState: {}
    name: ubuntu
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-08-01T15:52:58Z"
  hostIP: 192.168.49.2
  hostIPs:
  - ip: 192.168.49.2
  phase: Running
  podIP: 10.244.0.20
  podIPs:
  - ip: 10.244.0.20
  qosClass: BestEffort
  startTime: "2024-08-01T15:52:52Z"
```

```bash
snoopy_kr@iMac Basic % kubectl apply -f wrong-replicaset.yaml
The ReplicaSet "ubuntu" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"not-match-with-selector", "env":"test"}: `selector` does not match template `labels`
```

## Deployment

### Deployment 형식

- replicas - 유지하고자 하는 Pod의 갯수이다. 명시한 갯수만큼 정상적으로 동작하는 Pod의 갯수를 유지시킨다. (실행하려는 Pod가 정상적이지 않는 경우에 불가피하게 정상동작하는 Pod의 갯수를 유지하지 못하는 경우가 발생하기도 한다.)

- selector - 관리하고자 하는 Pod를 선택하기 위한 조건을 정의하는 필드이다. selector에는 다양한 조건들의 추가가 가능한데 방법에 대해서는 Label selectors 에서 확인 가능하다.

- template - 관리되고자 하는 Pod에 대해 정의하는 필드이다. Pod를 정의할때와 마찬가지로 metadata와 spec에 대해 정의해주면 된다.

```bash
snoopy_kr@iMac Basic % kubectl apply -f deployment.yaml
deployment.apps/nginx created

snoopy_kr@iMac Basic % kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/3     3            0           8s

snoopy_kr@iMac Basic % kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
nginx-59b5f565ff   3         3         3       21s

snoopy_kr@iMac Basic % kubectl get replicasets nginx-59b5f565ff -o yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    deployment.kubernetes.io/desired-replicas: "3"
    deployment.kubernetes.io/max-replicas: "4"
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-08-01T15:59:11Z"
  generation: 1
  labels:
    app: nginx
    pod-template-hash: 59b5f565ff
  name: nginx-59b5f565ff
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: nginx
    uid: ba9e3ec2-674e-4de8-9f0e-c6e4ade143c8
  resourceVersion: "5347"
  uid: 317b0eaa-53b7-4a1c-a797-3cd503da7f5a
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      pod-template-hash: 59b5f565ff
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
        pod-template-hash: 59b5f565ff
    spec:
      containers:
      - image: nginx:1.20.2
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 3
  fullyLabeledReplicas: 3
  observedGeneration: 1
  readyReplicas: 3
  replicas: 3

snoopy_kr@iMac Basic % kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-59b5f565ff-7xbfx   1/1     Running   0          74s
nginx-59b5f565ff-bd6d8   1/1     Running   0          74s
nginx-59b5f565ff-fgnjj   1/1     Running   0          74s
```

## Update 방식

### Recreate

Recreate 방식은 Deployment가 관리중인 모든 Pod를 없앤후에 새로운 Pod를 배포하는 방식이다. 따라서 불가피하게 Application이 모두 중단되는 downtime이 발생하게 된다.

```bash
snoopy_kr@iMac Basic % kubectl apply -f deployment-recreate.yaml
deployment.apps/nginx configured

snoopy_kr@iMac Basic % kubectl set image deployment/nginx nginx=nginx:1.21.6
deployment.apps/nginx image updated

snoopy_kr@iMac Basic % kubectl get pods                                   
NAME                     READY   STATUS              RESTARTS   AGE
nginx-745567d4b8-4zf57   0/1     ContainerCreating   0          7s
nginx-745567d4b8-cpxhm   0/1     ContainerCreating   0          7s
nginx-745567d4b8-nrpvv   0/1     ContainerCreating   0          7s
```

### RollingUpdate

RollingUpdate 방식은 기존 Pod를 일부 제거하고 새롭게 Update된 Pod를 일부 배포하는 과정을 반복하는 방식으로 점진적인 배포과정 때문에 완료까지 시간이 더 걸리겠지만 일부의 Pod는 계속 running 상태로 유지가 되기 때문에 Downtime이 발생하지 않는다.

- strategy.rollingUpdate.maxSurge - Update시 최대 얼마만큼의 Pod를 더 생성할 수 있는지 정할 수 있는 설정이다. int 값(ex: 5) 또는 string 값(ex: 20%)으로 %의 사용이 가능하다.

- strategy.rollingUpdate.maxUnavailable - Update시 최대 얼마만큼의 Pod가 unavailable 상태여도 되는지 정할 수 있는 설정이다. int 값(ex: 5) 또는 string 값(ex: 20%)으로 %의 사용이 가능하다.

```bash
snoopy_kr@iMac Basic % kubectl apply -f deployment.yaml               
deployment.apps/nginx created

snoopy_kr@iMac Basic % kubectl get pods                
NAME                     READY   STATUS    RESTARTS   AGE
nginx-59b5f565ff-d2crb   1/1     Running   0          9s
nginx-59b5f565ff-hgpsn   1/1     Running   0          9s
nginx-59b5f565ff-snpfq   1/1     Running   0          9s

snoopy_kr@iMac Basic % kubectl apply -f deployment-rolling-update.yaml 
deployment.apps/nginx configured

snoopy_kr@iMac Basic % kubectl get pods                               
NAME                     READY   STATUS    RESTARTS   AGE
nginx-59b5f565ff-d2crb   1/1     Running   0          22s
nginx-59b5f565ff-hgpsn   1/1     Running   0          22s
nginx-59b5f565ff-jtzkc   1/1     Running   0          2s
nginx-59b5f565ff-snpfq   1/1     Running   0          22s

snoopy_kr@iMac Basic % kubectl set image deployment/nginx nginx=nginx:1.21.6
deployment.apps/nginx image updated

snoopy_kr@iMac Basic % kubectl get pods                                     
NAME                     READY   STATUS              RESTARTS   AGE
nginx-59b5f565ff-d2crb   1/1     Running             0          76s
nginx-59b5f565ff-snpfq   1/1     Running             0          76s
nginx-745567d4b8-fncfg   0/1     ContainerCreating   0          1s
nginx-745567d4b8-ggr7r   0/1     ContainerCreating   0          1s
nginx-745567d4b8-m8pqn   0/1     ContainerCreating   0          1s
nginx-745567d4b8-xmgnf   0/1     ContainerCreating   0          1s

snoopy_kr@iMac Basic % kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-745567d4b8-fncfg   1/1     Running   0          6s
nginx-745567d4b8-ggr7r   1/1     Running   0          6s
nginx-745567d4b8-m8pqn   1/1     Running   0          6s
nginx-745567d4b8-xmgnf   1/1     Running   0          6s
```

## 롤백

```bash
snoopy_kr@iMac Basic % kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-59b5f565ff   0         0         0       103s
nginx-745567d4b8   4         4         4       28s

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=745567d4b8
  Containers:
   nginx:
    Image:      nginx:1.21.6
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>


snoopy_kr@iMac Basic % kubectl rollout undo deployment/nginx --to-revision=1

snoopy_kr@iMac Basic % kubectl rollout undo deployment/nginx
deployment.apps/nginx rolled back

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=745567d4b8
  Containers:
   nginx:
    Image:      nginx:1.21.6
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

snoopy_kr@iMac Basic % kubectl get rs                                       
NAME               DESIRED   CURRENT   READY   AGE
nginx-59b5f565ff   4         4         4       5m36s
nginx-745567d4b8   0         0         0       4m21s

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx --revision=3
deployment.apps/nginx with revision #3
Pod Template:
  Labels:       app=nginx
        pod-template-hash=59b5f565ff
  Containers:
   nginx:
    Image:      nginx:1.20.2
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

```bash
snoopy_kr@iMac Basic %  kubectl annotate deployment/nginx kubernetes.io/change-cause="Nginx version 1.20.2"
deployment.apps/nginx annotated

snoopy_kr@iMac Basic % kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         <none>
3         Nginx version 1.20.2
```

## File List

### single-container.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-container-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 1; done"]
```

### multiple-containers.yaml 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multiple-containers-pod
spec:
  containers:
  - name: ubuntu1
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello1; sleep 1; done"]
  - name: ubuntu2
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello2; sleep 1; done"]
```

### volume-share-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-pod
spec:
  containers:
  - name: ubuntu1
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello1 >> /ubuntu1/test.log; sleep 5; done"]
    volumeMounts:
    - mountPath: /ubuntu1
      name: test-volume
  - name: ubuntu2
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "tail -f /ubuntu2/test.log"]
    volumeMounts:
    - mountPath: /ubuntu2
      name: test-volume
  volumes:
  - name: test-volume
    emptyDir: {}
```

### network-share-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-share-pod
spec:
  containers:
  - name: spring-boot
    image: gentledev10/spring-boot-docker
  - name: ubuntu-curl
    image: gentledev10/ubuntu-curl
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(curl -s localhost:8080); sleep 5; done"]
```

### replicaset.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: ubuntu
  labels:
    env: test
    app: ubuntu
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        env: test
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo hello; sleep 1; done"]
```

### manual-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
  labels:
    app: ubuntu
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 1; done"]
```

### wrong-replicaset.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: ubuntu
  labels:
    env: test
    app: ubuntu
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        env: test
        app: not-match-with-selector
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo hello; sleep 1; done"]
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.2
```

### deployment-recreate.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.2
```

### deployment-rolling-update.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.2
```