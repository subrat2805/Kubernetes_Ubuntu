# k8s-resource

# Kubernetes Cluster installation using kubeadm
Follow this documentation to set up a Kubernetes cluster on **Ubuntu** machines.

This documentation guides you in setting up a cluster with one master node and two worker nodes.

## Prerequisites: 
1. System Requirements 
    >Master: t2.medium (2 CPUs and 2GB Memory)   
    >Worker Nodes: t2.medium 

1. Open the Below ports in the Security Group and launch all the instances with the same security groups 
   #### Master node: 
    `6443  
    32750  
    10250  
    4443  
    443  
    8080 
    179`

   ##### On Worker node:
    `179`  

   ### `On Both Master and Worker:`
1. Perform all the commands as root user unless otherwise specified
 
   Install, Enable and start docker service.
   Use the Docker repository to install docker.
   > While creating instances in AWS select the Ubuntu server version 20.04

   ```sh
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose

   ```
1. Start Docker services 
   ```sh
   sudo systemctl enable docker
   sudo systemctl start docker
   ```
1. Disable AppArmor
   ```sh
   sudo systemctl stop apparmor
   sudo systemctl disable apparmor
   sudo sed -i 's/^GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash apparmor=0"/' /etc/default/grub
   sudo update-grub
   ```
1. Disable Uncomplicated Firewall
   ```sh
   sudo systemctl disable ufw
   sudo systemctl stop ufw
   ```
1. Disable swap
   ```sh
    sudo sed -i '/swap/d' /etc/fstab
    sudo swapoff -a
   ```
1. Update sysctl settings for Kubernetes networking
   ```sh
   sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sysctl --system
   ```
## Kubernetes Setup
1. Add yum repository for kubernetes packages 
    ```sh
    sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B53DC80D13EDEF05
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg
    sudo apt-get update
    ```
1. Install Kubernetes
    ```sh
    sudo apt-get update
    sudo apt-get install -y kubeadm kubelet kubectl
    ```
1. Enable and Start kubelet service
    ```sh
    systemctl enable kubelet
    systemctl start kubelet
    ```
## `On Master Node Only:`
1. Initialize Kubernetes Cluster
    ```sh
    sudo kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
    ```

# `RUN-TIME ERROR HANDLE:`
1. During initializing the kubedam if you encounter any error, then by using the blow command you can handle that error
    ```sh
    sudo rm -rf /etc/containerd/config.toml
    sudo systemctl restart containerd
    ```
1. Copy kube config file.   
    ``To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.``  
    ```sh
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1. Deploy Calico network. 
	> This should be executed as a user where config file is copied
    
    ```sh
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
    ```

1. Cluster join command
    ```sh
    kubeadm token create --print-join-command
    ```
## `On Worker Node Only:`
1. Add worker nodes to cluster 
    > Use the output from __kubeadm token create__ command in previous step from the master server and run here.


## `On Master Node:`

1. Verifying the cluster
    To Get Nodes status
    ```sh
    kubectl get nodes
    ```
    To Get component status
    ```sh
    kubectl get cs
    ```




----------------------------------------
# For label change
 kubectl label node ip-172-31-86-115.ec2.internal kubernetes.io/role=worker --overwrite=true
