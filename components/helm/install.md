# Quickly install Helm

The package manager for Kubernetes.Helm is the best way to find, share, and use software built for Kubernetes.

## INSTALL

```
wget https://kubernetes-helm.storage.googleapis.com/helm-v2.11.0-linux-amd64.tar.gz

tar -zxvf helm-v2.11.0-linux-amd64.tar.gz && cd linux_

cp helm tiller /usr/local/bin/

helm init --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.10.0
```

## CHECK

```
kubectl get pods -n kube-system | grep tiller
```

## TILLER AND ROLE-BASED ACCESS CONTROL 

```
kubectl create serviceaccount --namespace=kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace=kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

## See also
https://docs.helm.sh/using_helm/#installing-helm
