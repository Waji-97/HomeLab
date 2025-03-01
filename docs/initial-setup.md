# Initial Setup (Getting Started)
I have 4 nodes in my HomeLab
| Device                      | Name      | Disk Size |  Ram  | CPU        | Operating System  | Purpose              |
|-----------------------------|-----------|-----------|-------|------------|-------------------|----------------------|
| Custom Made PC              | rel       | 1TB NVMe  | 64GB  | 24C/32T    |Ubuntu 22.04.5 LTS | Kubernetes Worker    |
| Mini PC 1                   | mini1     | 64GB SSD  | 16GB  | 4C/4T      |Ubuntu 22.04.5 LTS | Kubernetes Master 1  |
| Mini PC 2                   | mini2     | 64GB SSD  | 16GB  | 4C/4T      |Ubuntu 22.04.5 LTS | Kubernetes Master 2  |
| Mini PC 3                   | mini3     | 64GB SSD  | 16GB  | 4C/4T      |Ubuntu 22.04.5 LTS | Kubernetes Master 3  |

---

<br>

## Setting up the Cluster
I am using Kubespray to deploy Kubernetes in my HomeLab. My custom variables for the cluster & hosts (inventory) file can be found under `kubernetes/kubespray`

### Pre Settings for all nodes
Before running kubespray ansible, all of the nodes should be prepared with the following steps: 
- Create a root password (same for all nodes in my case)
- Update `/etc/ssh/sshd_config` file and uncomment the `PermitRootLogin yes` option within the file (for root SSH into all nodes)
-  Restart sshd service using systemctl

### Running Kubespray 
Running the docker command for kubespray
```bash
$ docker run --rm -it --mount type=bind,source="$(pwd)"/hosts.yml,dst=/inventory --mount type=bind,source="$(pwd)"/vars.yml,dst=/vars quay.io/kubespray/kubespray:v2.27.0 bash
```
<br>

Once inside the Kubespray container, run the following ansible command to verify if the container can reach to all nodes via SSH
```bash
$ ansible all -i /inventory -m ping -k
SSH password: 
[WARNING]: Skipping callback plugin 'ara_default', unable to load
mini1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
mini2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
mini3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
rel | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
<br>

Finally, run the kubespray `cluster.yml` file to install kubernetes
```bash
$ ansible-playbook -e @/vars -i /inventory cluster.yml -k
```

<br>


### Verify Kubernetes Cluster
After successful installation, verify the cluster
```bash
## get nodes
➜  ~ k get no
NAME    STATUS   ROLES           AGE   VERSION
mini1   Ready    control-plane   12m   v1.31.4
mini2   Ready    control-plane   12m   v1.31.4
mini3   Ready    control-plane   12m   v1.31.4
rel     Ready    <none>          11m   v1.31.4

## get all pods
➜  ~ k get po -A
NAMESPACE     NAME                                                READY   STATUS    RESTARTS        AGE
argocd        argocd-application-controller-0                     1/1     Running   0               8m46s
argocd        argocd-applicationset-controller-865d876d77-4rdlr   1/1     Running   0               8m47s
argocd        argocd-dex-server-66459bdb8f-4f8qd                  1/1     Running   1 (6m56s ago)   8m47s
argocd        argocd-notifications-controller-78794ddcb5-kdj95    1/1     Running   0               8m47s
argocd        argocd-redis-8846c4d6c-l8vh8                        1/1     Running   0               8m47s
argocd        argocd-repo-server-5bc77b85cd-xv9qx                 1/1     Running   0               8m47s
argocd        argocd-server-847bbc55d9-sxzdk                      1/1     Running   0               8m47s
kube-system   calico-kube-controllers-5db5978889-xvzm5            1/1     Running   0               9m45s
kube-system   calico-node-625k2                                   1/1     Running   0               10m
kube-system   calico-node-cp85h                                   1/1     Running   0               10m
kube-system   calico-node-jqsqj                                   1/1     Running   0               10m
kube-system   calico-node-k9zvs                                   1/1     Running   0               10m
kube-system   coredns-d665d669-f85z7                              1/1     Running   0               9m35s
kube-system   coredns-d665d669-rj4wv                              1/1     Running   0               9m31s
kube-system   dns-autoscaler-5cb4578f5f-zsqtx                     1/1     Running   0               9m32s
kube-system   etcd-mini1                                          1/1     Running   0               13m
kube-system   etcd-mini2                                          1/1     Running   0               13m
kube-system   etcd-mini3                                          1/1     Running   0               13m
kube-system   kube-apiserver-mini1                                1/1     Running   1               13m
kube-system   kube-apiserver-mini2                                1/1     Running   1               13m
kube-system   kube-apiserver-mini3                                1/1     Running   1               13m
kube-system   kube-controller-manager-mini1                       1/1     Running   2               13m
kube-system   kube-controller-manager-mini2                       1/1     Running   2               13m
kube-system   kube-controller-manager-mini3                       1/1     Running   2               13m
kube-system   kube-proxy-7dtms                                    1/1     Running   0               13m
kube-system   kube-proxy-8ntd5                                    1/1     Running   0               13m
kube-system   kube-proxy-pc5vn                                    1/1     Running   0               13m
kube-system   kube-proxy-sl5q7                                    1/1     Running   0               11m
kube-system   kube-scheduler-mini1                                1/1     Running   1               13m
kube-system   kube-scheduler-mini2                                1/1     Running   1               13m
kube-system   kube-scheduler-mini3                                1/1     Running   1               13m
kube-system   kube-vip-mini1                                      1/1     Running   1 (10m ago)     13m
kube-system   kube-vip-mini2                                      1/1     Running   0               13m
kube-system   kube-vip-mini3                                      1/1     Running   1 (8m8s ago)    13m
kube-system   nodelocaldns-4fqf6                                  1/1     Running   0               9m27s
kube-system   nodelocaldns-gbfn7                                  1/1     Running   0               9m27s
kube-system   nodelocaldns-lnvl2                                  1/1     Running   0               9m27s
kube-system   nodelocaldns-qv9pf                                  1/1     Running   0               9m27s

```