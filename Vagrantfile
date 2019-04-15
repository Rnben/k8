# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF8"

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: $script
  config.vm.define "master" do |master|
    master.vm.box = "bento/centos-7.4"
    master.vm.hostname = "master"
    master.ssh.username = "root"
    master.ssh.password = "vagrant"
    master.ssh.insert_key = true
    master.vm.box_check_update = false
    master.vm.network "private_network", ip: "11.11.11.101"
    master.vm.provider "virtualbox" do |vb|
          vb.customize ["modifyvm", :id, "--name", "master", "--memory", "2048"]
          vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
  end
  config.vm.define "node" do |node|
    node.vm.box = "bento/centos-7.4"
    node.vm.hostname = "node"
    node.ssh.username = "root"
    node.ssh.password = "vagrant"
    node.ssh.insert_key = true
    node.vm.box_check_update = false
    node.vm.network "private_network", ip: "11.11.11.102"
    node.vm.provider "virtualbox" do |vb|
          vb.customize ["modifyvm", :id, "--name", "node", "--memory", "1024"]
          vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
end

$script = <<-SCRIPT
# 关闭swap
swapoff -a && sed -i 's/.*swap.*/#&/' /etc/fstab

# 禁用selinux
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux && setenforce 0

# 开启forward
iptables -P FORWARD ACCEPT

# 配置转发相关参数
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
sysctl --system

# hosts配置
cat <<EOF > /etc/hosts
11.11.11.101 master
11.11.11.102 node
EOF

# aliyun
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache

# docker
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

# keepalived、haproxy
yum install -y socat keepalived ipvsadm haproxy

# kubernetes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl ebtables wget vim

cat <<EOF >/etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
EOF

systemctl start kubelet
systemctl start docker
systemctl enable docker
systemctl enable kubelet

cat > /root/kubeadm-master.config <<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
noTaintMaster: true
kubernetesVersion: v1.11.2
api:
  advertiseAddress: 11.11.11.101
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kubeProxy:
  config:
    mode: ipvs
networking:
  podSubnet: 10.244.0.0/16
EOF

curl https://raw.githubusercontent.com/Rnben/k8s-kubeadm/master/flannel.yaml -o /root/flannel.yaml

cat >> /etc/profile <<EOF
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

a=$(lsmod|grep ip_vs)
echo $a
SCRIPT
