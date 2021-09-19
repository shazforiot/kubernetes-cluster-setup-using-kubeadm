# kubernetes-cluster-setup-using-kubeadm
How to create kubernetes cluster using kubeadm

Follow the below steps for creating Kubernetes Cluster on CentOs.

Steps Involved:

1. Set Hostnames
2. Assign Static IP
3. Edit /etc/hosts file
4. Disable SELinux
5. Disable firewall and edit Iptables settings
6. Setup Kubernetes Repo
7. Installing Kubeadm and Docker, Enable and start the services
8. Disable Swap
9. Initialize Kubernetes Cluster
10. Installing Pod Network using Calico network
11. Join Worker Nodes

Steps 1 tp 8 is done on Both Master and worker nodes, Steps 9 & 10 is to be done only on master node, step 11 is done only on worker nodes.

-------------------------------------------------------

**Set Hostnames**

hostnamectl set-hostname k8smaster (On Master)
hostnamectl set-hostname k8sworker1(On Node1)
hostnamectl set-hostname k8sworker2 (On Node2)

------------------------------------------------------

**Assign Static IP**

Run nmcli con to indentify the network details.

Go to vi /etc/sysconfig/network-scripts/ and change the settings in ifcfg-ens33 (the name will change based on your network device name) as below. 

Below is a sample format 

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
IPADDR=XXX.XXX.XXX.XXX
PREFIX=24
GATEWAY=XXX.XXX.XXX.XXX
DNS1=192.168.2.254
DNS2=8.8.8.8
DNS3=8.8.4.4
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_AUTOCONF="no"
IPV6_DEFROUTE="no"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="Your respective network UUID"
DEVICE="ens33"
ONBOOT="yes"

Run the command  systemctl restart network to restart the network

------------------------------------------------------------

**Edit /etc/hosts file**

Run the below commands on the machines. Change the IP address and host name as per your machine settings.
cat <<EOF>> /etc/hosts

192.168.0.xxx k8smaster
192.168.0.xxx k8sworker1
192.168.0.xxx k8sworker2

EOF
  
-----------------------------------------------------------
  
**Disable SELinux**
  
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

-----------------------------------------------------------
  
**Disable firewall and edit Iptables settings**
  
systemctl disable firewalld
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
  
----------------------------------------------------------
  
**Setup Kubernetes Repo**

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

---------------------------------------------------------
  
**Installing Kubeadm and Docker, Enable and start the services**
  
yum install kubeadm docker -y
systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker
  
--------------------------------------------------------
  
**Disable Swap**
  
swapoff -a
vi /etc/fstab and Comment the line with Swap Keyword
  
-----------------------------------------------------
  
**Initialize Kubernetes Cluster**

kubeadm init
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
  
------------------------------------------------------
  
**Installing Pod Network using Calico network**


curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
kubectl get pods -n kube-system

----------------------------------------------------

  **Join Worker Nodes **
  
  Use the token from Kubeadmin init screen. Below is a sample how it looks like.
  
  kubeadm join 192.168.0.xxx:6443 --token XXX\
        --discovery-token-ca-cert-hash sha256:XX

  ---------------------------------------------------
