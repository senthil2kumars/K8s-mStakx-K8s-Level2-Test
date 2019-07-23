# k8s-install
Creating Highly Available clusters with kubeadm on AWS ubuntu 16.04 Instance

### Prerequisites

1) This example assumes that you have 1 Ansible, 1LB, 3 Master and N no. of worker node( Min 2 is recommended) with min 4CPU X 8 GBRAM servers.Here I have used t2.xlarge(4cpuX16 GBRAM) for Master Nodes, c5.xlarge(4CPU X 8GB RAM)for Worker Nodes and t2.micro (1CPU X 1GBRAM) for LB on AWS instance with ubuntu-16.04Lts for better performance and additonal applicaton deployment.See the [k8s installation doc.](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) for details about creating a cluster.
2) Install ansible package on Ansible node. See the [Ansible installation on ubuntu server](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu) 

### Install k8s

1. Before executing the ansible playbook, we need to update the Nodes IP address and sudo username and Pem key for authentication on the host file as example below.

    ```
    [allservers]
    <MASTER1 Node IP>	ansible_ssh_user=ubuntu
    <MASTER2 Node IP> 	ansible_ssh_user=ubuntu
    <MASTER3 Node IP> 	ansible_ssh_user=ubuntu
    <WORKER1 Node IP> 	ansible_ssh_user=ubuntu
    <WORKER2 Node IP>	ansible_ssh_user=ubuntu

    [allservers:vars]
    ansible_user=ubuntu
    ansible_ssh_private_key_file=/etc/ansible/<ssh key pem file>.pem
    ```

2. Use the `kubernetes-prereq.yaml` to configure the prerequesties on a cluster node. Which will initiate 1)Install Docker, 2) Install necessary Deb package 3) Install kubeadm, kubelet and kubectl and etc.
See the [Configure K8s prerequesties](https://github.com/senthil2kumars/K8s-mStakx-Level2-Test/tree/master/k8s-install-Prerequisites) 
    ```
        $ansible-playbook kubernetes-prereq.yaml
    ```
  Result: Ansible will configure required prerequesties on a cluster node.

3. Install the Highly Available clusters with kubeadm on AWS ubuntu 16.04 Instance. See the [Creating Highly Available clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) 

A)Steps for the first control plane node

  On the first control plane node, create a configuration file called kubeadm-config.yaml:
  
  
  ```
    cat >> kubeadm-config.yaml
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    kubernetesVersion: stable
    controlPlaneEndpoint: "LOAD_BALANCER_IP:6443"
    EOF
  ```
 B)Initialize the control plane:
    
 ```
    $kubeadm init --config=kubeadm-config.yaml --upload-certs
    .
    .
    .
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

    You can now join any number of the control-plane node running the following command on each as root:

    kubeadm join 172.31.87.214:6443 --token fpbfqf.rebejery8kjram12 \
    --discovery-token-ca-cert-hash sha256:6e13976095f5dcc4912eeb7f68a757df06fcf0cfe9051048409ab08044cc399a \
    --control-plane --certificate-key 045e46d1a3410be8c565b32b100c36ad4deffbb66296d326e2691d12256392b2

    Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
    As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use 
    "kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

    Then you can join any number of worker nodes by running the following on each as root:

    kubeadm join 172.31.87.214:6443 --token fpbfqf.rebejery8kjram12 \
    --discovery-token-ca-cert-hash sha256:6e13976095f5dcc4912eeb7f68a757df06fcf0cfe9051048409ab08044cc399a 
```
    
  
 Note: Copy this output to a text file. You will need it later to join control plane and worker nodes to the cluster.
 
 C)Apply the CNI plugin of your choice: Follow these instructions to install the CNI provider. Make sure the configuration      corresponds to the Pod CIDR specified in the kubeadm configuration file if applicable.

In this example we are using Weave Net:
    ```
  $kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```
 D) Login into other Master node and execute below
 ```
 $kubeadm join 172.31.87.214:6443 --token fpbfqf.rebejery8kjram12 \
    --discovery-token-ca-cert-hash sha256:6e13976095f5dcc4912eeb7f68a757df06fcf0cfe9051048409ab08044cc399a \
    --control-plane --certificate-key 045e46d1a3410be8c565b32b100c36ad4deffbb66296d326e2691d12256392b2
 ```
 E) Login into Worker nodes and execute below
 ```
 kubeadm join 172.31.87.214:6443 --token fpbfqf.rebejery8kjram12 \
    --discovery-token-ca-cert-hash sha256:6e13976095f5dcc4912eeb7f68a757df06fcf0cfe9051048409ab08044cc399a 
 ```
 F)Check the nodes staus and POd status
 ```
 root@ip-172-31-87-214:~# kubectl get nodes
NAME               STATUS   ROLES    AGE     VERSION
ip-172-31-35-11    Ready    master   7h45m   v1.15.1
ip-172-31-35-208   Ready    master   7h51m   v1.15.1
ip-172-31-35-224   Ready    <none>   7h43m   v1.15.1
ip-172-31-40-236   Ready    <none>   7h44m   v1.15.1
ip-172-31-46-242   Ready    master   7h46m   v1.15.1

root@ip-172-31-87-214:~# kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-5c98db65d4-6ccsr                   1/1     Running   1          7h52m
coredns-5c98db65d4-vd6k7                   1/1     Running   1          7h52m
etcd-ip-172-31-35-11                       1/1     Running   1          7h46m
etcd-ip-172-31-35-208                      1/1     Running   1          7h51m
etcd-ip-172-31-46-242                      1/1     Running   1          7h47m
kube-apiserver-ip-172-31-35-11             1/1     Running   1          7h46m
kube-apiserver-ip-172-31-35-208            1/1     Running   1          7h51m
kube-apiserver-ip-172-31-46-242            1/1     Running   1          7h47m
kube-controller-manager-ip-172-31-35-11    1/1     Running   1          7h46m
kube-controller-manager-ip-172-31-35-208   1/1     Running   2          7h51m
kube-controller-manager-ip-172-31-46-242   1/1     Running   1          7h47m
kube-proxy-5bnrs                           1/1     Running   1          7h52m
kube-proxy-b6s5p                           1/1     Running   1          7h45m
kube-proxy-brqzj                           1/1     Running   1          7h44m
kube-proxy-dnwqr                           1/1     Running   1          7h46m
kube-proxy-dwdfk                           1/1     Running   1          7h47m
kube-scheduler-ip-172-31-35-11             1/1     Running   1          7h46m
kube-scheduler-ip-172-31-35-208            1/1     Running   2          7h51m
kube-scheduler-ip-172-31-46-242            1/1     Running   1          7h47m
tiller-deploy-56c686464b-swncf             1/1     Running   1          7h32m
weave-net-6p89k                            2/2     Running   3          7h45m
weave-net-7tkfk                            2/2     Running   3          7h50m
weave-net-9z899                            2/2     Running   3          7h46m
weave-net-kgx86                            2/2     Running   4          7h47m
weave-net-qn4mw                            2/2     Running   3          7h44m

```
