Title:___How to setup a kubernetes dashboard___
Date:____09.03.2021__
Author:__Gabriel Mendoza Reyes__
Keyword: Cluster, Distributed Computing, kubernetes, kubespray, Reverse Proxy

# Introduction
The kubernetes clusters dashboard is not accessable out of the box. When accessing the address given by `kubectl cluster-info` which is `https://10.0.20.101:6443` I get an https certificate warning, when accepting the warning I get json with a failure message.
This document describes my research and other aspects on how to solve the issue and get access to the dashboard. 

# Background 
According to [australtech]{https://www.australtech.net/kubernetes-unable-to-login-to-the-dashboard/} there are three ways of accessing the dashboard:

- kubectl proxy
- NodePort
- API Server 

The Api Server solution is the recommended one, since `kubectl proxy` would expose a non secure http connection and the `NodePort` only works with a single master node, which does not suite our HA-setup. 

# How To
The kubectl proxy method is the easiest setup for the time being I will use this setup until I have setup the real cluster. 

- Install the dashboard:
    `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml`

- Load proxy
    `kubectl proxy`

- Create a Service Account
```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

- Create a ClusterRoleBinding

```shell
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```
- Get the Bearer Token


```shell
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

- Log onto Dashboard:
[link]{http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/}