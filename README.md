# k8s-resource

# Kubernetes Cluster installation using kubeadm
Follow this documentation to set up a Kubernetes cluster on **REDHAT** machines.

This documentation guides you in setting up a cluster with one master node and two worker nodes.

## Prerequisites: 
1. System Requirements 
    >Master: t2.medium (2 CPUs and 2GB Memory)   
    >Worker Nodes: t2.micro 

1. Open Below ports in the Security Group. 
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
   > If you use docker from CentOS OS repository, the docker version might be old to work with Kubernetes v1.13.0 and above

   ```sh
   sudo yum install -y yum-utils
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```
1. Start Docker services 
   ```sh
   systemctl enable docker
   systemctl start docker
   ```
1. Disable SELinux
   ```sh
   setenforce 0
   sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
   ```
1. Disable Firewall
   ```sh
   systemctl disable firewalld
   systemctl stop firewalld
   ```
1. Disable swap
     ```sh
     sed -i '/swap/d' /etc/fstab
     swapoff -a
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
    sudo tee /etc/yum.repos.d/kubernetes.repo<<EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
    ```
1. Install Kubernetes
    ```sh
    yum install -y kubeadm kubelet kubectl
    ```
1. Enable and Start kubelet service
    ```sh
    systemctl enable kubelet
    systemctl start kubelet
    ```
## `On Master Node Only:`
1. Initialize Kubernetes Cluster
    ```sh
    kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
    ```
1. Create a user for kubernetes administration  and copy kube config file.   
    ``To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.``  
    In this case, we are creating a user called `kubeadmin`
    ```sh
    useradd kubeadmin 
    mkdir /home/kubeadmin/.kube
    cp /etc/kubernetes/admin.conf /home/kubeadmin/.kube/config
    chown -R kubeadmin:kubeadmin /home/kubeadmin/.kube
    ```
1. Deploy Calico network as a __kubeadmin__ user. 
	> This should be executed as a user (heare as a __kubeadmin__ )
    
    ```sh
    sudo su - kubeadmin 
    curl https://docs.projectcalico.org/manifests/calico.yaml -o calico.yaml
    kubectl apply -f calico.yaml
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


# RUN-TIME ERROR HANDLE

rm -rf /etc/containerd/config.toml

systemctl restart containerd

----------------------------------------
# For label change
 kubectl label node ip-172-31-86-115.ec2.internal kubernetes.io/role=worker --overwrite=true
