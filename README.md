# kubernetes-cluster-setup-using-kubeadm
How to create kubernetes cluster using kubeadm

Watch the video :https://youtu.be/cAZ5nkLfL6M

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

hostnamectl set-hostname k8smaster (On Master)<br />
hostnamectl set-hostname k8sworker1(On Node1)<br />
hostnamectl set-hostname k8sworker2 (On Node2)<br />

------------------------------------------------------

**Assign Static IP**

Run nmcli con to indentify the network details.<br />

Go to vi /etc/sysconfig/network-scripts/ and change the settings in ifcfg-ens33 (the name will change based on your network device name) as below. <br />

Below is a sample format <br />

TYPE="Ethernet"<br />
PROXY_METHOD="none"<br />
BROWSER_ONLY="no"<br />
BOOTPROTO="none"<br />
IPADDR=XXX.XXX.XXX.XXX<br />
PREFIX=24<br />
GATEWAY=XXX.XXX.XXX.XXX<br />
DNS1=192.168.2.254<br />
DNS2=8.8.8.8<br />
DNS3=8.8.4.4<br />
DEFROUTE="yes"<br />
IPV4_FAILURE_FATAL="no"<br />
IPV6INIT="no"<br />
IPV6_AUTOCONF="no"<br />
IPV6_DEFROUTE="no"<br />
IPV6_FAILURE_FATAL="no"<br />
IPV6_ADDR_GEN_MODE="stable-privacy"<br />
NAME="ens33"<br />
UUID="Your respective network UUID"<br />
DEVICE="ens33"<br />
ONBOOT="yes"<br />

Run the command  systemctl restart network to restart the network<br />

------------------------------------------------------------

**Edit /etc/hosts file**

Run the below commands on the machines. Change the IP address and host name as per your machine settings.<br />
cat << EOF >> /etc/hosts<br />

192.168.0.xxx k8smaster<br />
192.168.0.xxx k8sworker1<br />
192.168.0.xxx k8sworker2<br />

EOF
  
-----------------------------------------------------------
  
**Disable SELinux**
  
setenforce 0<br />
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux<br />

-----------------------------------------------------------
  
**Disable firewall and edit Iptables settings**
  
systemctl disable firewalld<br />
modprobe br_netfilter<br />
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables<br />
  
----------------------------------------------------------
  
**Setup Kubernetes Repo**

cat << EOF > /etc/yum.repos.d/kubernetes.repo<br />
[kubernetes]<br />
name=Kubernetes<br />
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64<br />
enabled=1<br />
gpgcheck=1<br />
repo_gpgcheck=1<br />
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg<br />
EOF<br />

---------------------------------------------------------
  
**Installing Kubeadm and Docker, Enable and start the services**
  
yum install kubeadm docker -y<br />
systemctl enable kubelet<br />
systemctl start kubelet<br />
systemctl enable docker<br />
systemctl start docker<br />
  
--------------------------------------------------------
  
**Disable Swap**
  
swapoff -a<br />
vi /etc/fstab and Comment the line with Swap Keyword<br />
  
-----------------------------------------------------
  
**Initialize Kubernetes Cluster**

kubeadm init<br />
mkdir -p $HOME/.kube<br />
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config<br />
chown $(id -u):$(id -g) $HOME/.kube/config<br />
  
------------------------------------------------------
  
**Installing Pod Network using Calico network**


curl https://docs.projectcalico.org/manifests/calico.yaml -O<br />
kubectl apply -f calico.yaml<br />
kubectl get pods -n kube-system<br />

----------------------------------------------------

  **Join Worker Nodes **
  
  Use the token from Kubeadmin init screen. Below is a sample how it looks like.<br />
  
  kubeadm join 192.168.0.xxx:6443 --token XXX\
        --discovery-token-ca-cert-hash sha256:XX<br />

  ---------------------------------------------------
