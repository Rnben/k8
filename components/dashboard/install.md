## Deploy dashboard
1. `wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml`

2. Change ClusterIP to NodePort

3. `kubectl apply -f kubernetes-dashboard.yaml`

4. `kubectl -n kube-system get svc kubernetes-dashboard`

## Accessing dashboard

### Option 1 Use Token
1. Create token
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```
2. `kubectl -n kube-system get secret|grep admin-token`

3. `kubectl -n kube-system describe secret admin-token-nwphb`

### Option 2 Use kubeconfig