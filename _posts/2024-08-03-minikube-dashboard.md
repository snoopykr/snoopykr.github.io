---
layout: single
title: "Minikube Dashboard"
categories : Kubernetes
tag: [Kubernetes, Minikube, K9s, Dashboard]
---

Kubernetes Dashboard & K9s

## Dashboard

### Dashboard 준비

```bash
snoopy_kr@iMac Dashboard % kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml

namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

### Dashboard 확인

```bash
snoopy_kr@iMac Dashboard % kubectl get deployment -n kubernetes-dashboard
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
dashboard-metrics-scraper   1/1     1            1           47s
kubernetes-dashboard        1/1     1            1           48s
```

### Dashboard 실행

```bash
snoopy_kr@iMac Dashboard % kubectl proxy
Starting to serve on 127.0.0.1:8001
```

### Dashboard token 생성

쿠버네티스에서는 권한을 Role로 관리해주게 되는데, 이러한 Role을 부여해주는 행위를 Role Binding이라고 한다.

쿠버네티스에는 기본적으로 제공하는 여러 Role이 있으며 그중에서 Cluster Role은 Cluster에 대한 권한을 의미하는 특별한 의미의 Role이다.

```bash
snoopy_kr@iMac Dashboard % kubectl apply -f svc.yaml 
serviceaccount/dashboard-admin created

snoopy_kr@iMac Dashboard % kubectl apply -f cluster-role-binding.yaml 
clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created

snoopy_kr@iMac Dashboard % kubectl -n kubernetes-dashboard create token dashboard-admin
eyJhbGciOiJSUzI1NiIsImtpZCI6Ik5DNGFKbUlnTWhVQkFnaS1MaTMwREVvMGpzX0VrSnptajFGUE96Y0RPNncifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzIyNTkyNDkwLCJpYXQiOjE3MjI1ODg4OTAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiYTE1YmNlZDAtY2UwYi00YmU2LWFkMjMtNDVjODE4ZDVmMGUxIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJkYXNoYm9hcmQtYWRtaW4iLCJ1aWQiOiJmZDcyNmE0ZS0yM2QxLTQ3ODMtOWRkMS05OGZkOTY3OThmN2MifX0sIm5iZiI6MTcyMjU4ODg5MCwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.FEKLgbPk20hbd0p9j7zXPAC52klaSPaxYMllF1EjOpOTefi2vf9LaaiK-QlJ4sb5an3mFFbXY4kicrOYAV4Plebm7rYoG5xQo6UICDr1poksv-7Wxbx77B3LhDNNH3o3Xsx6ZgFbxH4j_-1Xn7XSfmFvkZU8ucphUbzhf-dxRh4IbGsys4NWR7Twz670r7vYdI7lPE4seNtwXwvtQUOg7sELkiY05hr0yYj_BnUurfg_MBdW2EWSqtMu0EzZxCP23310RY9uqh9rDgB-J4GRSyCwm8V_EBlTFKOeaQfZQPksVoqqT_649tnv0GbUSxR4ngdTlDUpvdCkWG1QYM77Yw
```

### Dashboard login

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

### svc.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
```

### cluster-role-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
```

## K9s

### K9s 설치하기

https://github.com/derailed/k9s/releases

### 압축 해제 후 작업

```bash
snoopy_kr@iMac Downloads % chmod 755 k9s

snoopy_kr@iMac Downloads % sudo mv k9s /usr/local/bin
```

### K9s 실행

```bash
snoopy_kr@iMac Downloads % k9s
```

```
 Context: minikube                                 <0> all       <a>       Attach       <ctrl-k>  Kill            <o> Show Node          ____  __.________        
 Cluster: minikube                                 <1> default   <ctrl-d>  Delete       <l>       Logs            <f> Show PortForward  |    |/ _/   __   \______ 
 User:    minikube                                               <d>       Describe     <p>       Logs Previous   <t> Transfer          |      < \____    /  ___/ 
 K9s Rev: v0.32.5                                                <e>       Edit         <shift-f> Port-Forward    <y> YAML              |    |  \   /    /\___ \  
 K8s Rev: v1.30.0                                                <?>       Help         <z>       Sanitize                              |____|__ \ /____//____  > 
 CPU:     n/a                                                    <shift-j> Jump Owner   <s>       Shell                                         \/            \/  
 MEM:     n/a                                                                                                                                                     
┌─────────────────────────────────────────────────────────────────────── Pods(default)[3] ───────────────────────────────────────────────────────────────────────┐
│ NAME↑                             PF           READY           STATUS                       RESTARTS IP                    NODE                AGE             │
│ nginx-59b5f565ff-q2gvc            ●            1/1             Running                             0 10.244.0.6            minikube            17m             │
│ nginx-59b5f565ff-qp869            ●            1/1             Running                             0 10.244.0.5            minikube            17m             │
│ nginx-59b5f565ff-tc7v7            ●            1/1             Running                             0 10.244.0.7            minikube            17m             │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
│                                                                                                                                                                │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
  <pod>                                                                                                                                                           
```                                                                                                                          
