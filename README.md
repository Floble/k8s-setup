# k8s-setup

## Description
Ansible playbooks for setting up a k8s cluster in high availability setup.

## Prerequisites
### Nodes
* At least **one master/worker** node.
* For the **ha setup**, at least **one ha proxy** node.
* **SSH access** to the master/worker node.
* **Client host** or **remote host** (in the following named **commander node**) running **Ubuntu** and **Ansible**.

**Notes:** The current version of the playbooks requires **Ubuntu** to be also running on **all master** and **worker nodes**.

### Egress/Ingress
* The **master, worker**, and **commander nodes** require **access to the internet**.
    * The following **outbound traffic** must be **allowed** by your **firewall** for these nodes:
        
        | Type    | Protocol | Port Range | Destination |
        | --------|:--------:|-----------:|------------:| 
        | All TCP | TCP      | 0 - 65535  | 0.0.0.0/0   |

* For **services** that must be **publicly avaibable from the internet**, you must deploy a **load balancer/ingress controller** in front of your nodes and ensure that **incoming traffic is routed to the worker nodes** appropriately. E.g., with **amazon web services (AWS)**, you can use the **amazon elastic load balancing application load balancer (ALB)** and the **AWS ALB Ingress controller**.

### Networking Model
* You must ensure that all **pods can communicate with eath other without** network address translation **(NAT)** and **with the same IPs**.
    * The playbooks deploy an **overlay network** using **WeaveNet**. The following **inbound traffic** must be **allowed** by your **firewall** for the nodes:

        | Type    | Protocol | Port Range | Source                   |
        | --------|:--------:|-----------:|------------------------: | 
        | All TCP | TCP      | 0 - 65535  | \<CIDR for the pod IPs>  |

    * You can also **configure** the **underlying network**, so it is **aware of the pod IPs**. E.g., with **AWS** you can use the **container networking interface (CNI) plugin**.

### Ansible
* **Ansible** must be installed on the **commander node** (see https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-18-04).

    ```
    sudo apt update
    sudo apt install software-properties-common
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt update
    sudo apt install ansible
    ```

## Execution of the Playbooks
### Prepare the inventory
Set the **values** for the **variables** in the **inventory** files **hosts**.

| Variable                   | Description                                                                           |
| ---------------------------|:--------------------------------------------------------------------------------------|
| <ip_address_1(n)>          | The ip address of the master, worker, or ha proxy node                                |
| <user_1(n)>                | The user to login as on the specific node                                             |
| <path_to_your_private_key> | Path to your private SSH key                                                          |
| master_primary             | The master node that must be initiated first                                          |
| masters_secondary          | The master nodes that will be initiated based on the initiation of the primary master |
| <public_SSH_gateway>       | The bastion host for accessing the master, worker, and ha proxy nodes via SSH         |

### Run the playbooks
Run the playbooks via from the directory where they are placed.

```
ansible-playbook -i hosts setup.yml
```

For debug mode, run:

```
ansible-playbook -i hosts setup.yml -vvv
```

## Architecture
More details on the architecture of the Kubernetes cluster that is deploy with the playbooks will be provided here soon.

## Results
* All the Kubernetes **nodes** must be in **available** state.
* A **nginx** pod must be deployed in your Kubernetes cluster that is in **running** state.

## References
* https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-1-11-cluster-using-kubeadm-on-ubuntu-18-04
* https://blog.inkubate.io/install-and-configure-a-multi-master-kubernetes-cluster-with-kubeadm/