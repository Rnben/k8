# Setup

## Prerequisites
1. Download the Istio release
2. Kubernetes platform setup
3. Requirements for Pods and Services
4. Install the Helm client
5. Istio by default uses `LoadBalancer` service object types. Some platforms do not support `LoadBalancer` service objects. For platforms lacking `LoadBalancer` support, install Istio with `NodePort` support instead with the flags --set gateways.istio-ingressgateway.type=`NodePort` --set gateways.istio-egressgateway.type=`NodePort` appended to the end of the Helm operation.

## Installation steps
1. Download the Istio release page
```
wget https://github.com/istio/istio/releases/download/1.0.4/istio-1.0.4-linux.tar.gz
```
2. Move to the Istio package directory
```
cd istio-1.0.3
```

3. Add the istioctl client to PATH
```
export PATH=$PWD/bin:$PATH
```

4. Install with Helm via helm template
```
helm template install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml

kubectl create namespace istio-system
kubectl apply -f $HOME/istio.yaml
```

5. Uninstall
```
kubectl delete -f $HOME/istio.yaml
```

## Deploying example app
```
# Bring up the application containers
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)

# Confirm all services and pods are correctly defined and running
kubectl get services
kubectl get pods

# Determining the ingress IP and port
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

kubectl get gateway

kubectl get svc istio-ingressgateway -n istio-system

# Proceed to Confirm the app is running
curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
```

## See so
https://istio.io/docs/examples/bookinfo/
