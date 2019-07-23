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
