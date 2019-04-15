# Creating a single-master-cluster

> two step to create a single master cluster
>* create vm
>* kubeadm init

## VM Configuration

| os  | kubernetes | CPU | MEM |
|---|---|---|---| 
| centos7.4 | 1.11.2 | 2C | 4G |

## Prerequisites
1. Install brew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Install vagrant and virtualbox

```
brew cask install vagrant
brew cask install virtualbox
```

## Create vm

3. Run this cmd in directory with Vagrantfile
```
vagrant up
```

## Create kubernetes cluster

4. Init kubernetes master
```
kubeadm init --config=kubeadm.config

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

5. Remove master taint

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

6. Install flannel
```
kubectl create -f flannel.yaml
```

7. Check cluster

```
kubectl cluster-info
kubectl get cs
kubectl get nodes
```

8. Join node

**Option 1**
```
# join node
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

# list token
kubeadm token list

# discovery-token-ca-cert-hash
root@ubuntu:/home/cong# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

```
**Option 2**
```
kubeadm token create --print-join-command
```

9. delete node
```
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```




