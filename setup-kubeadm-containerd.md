# Install Kubernetes with Kubeadm and Containerd

## 1. Prerequisites
* 2 GB or more of RAM per machine (any less will leave little room for your apps).
* 2 CPUs or more.
* Full network connectivity between all machines in the cluster (public or private network is fine).
* Unique hostname, MAC address, and product_uuid for every node.
* Certain ports are open on your machines.

## Do this on both kMaster and kWorker:

* Diable Swap:
    ```bash
    swapoff -a; sed -i '/swap/d' /etc/fstab
    ```

### 2. Install Containerd Engine

* Install pre-requisites for containerd
    ```bash
    sudo apt-get update \
        && sudo apt-get install -y apt-transport-https \
            ca-certificates curl software-properties-common
    ```

* Load modules overlay and br_netfilter
    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay \
        && sudo modprobe br_netfilter
    ```

* Set sysctl parameters and reload conf files
    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF

    sudo sysctl --system
    ```

* Setup the Docker repo
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -

    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```

* Install containerd
    ```bash
    sudo apt-get update \
    && sudo apt-get install -y containerd.io
    ```

* Create default config.toml
    ```bash
    sudo mkdir -p /etc/containerd \
    && sudo containerd config default | sudo tee /etc/containerd/config.toml
    ```

* Configure containerd to use systemd cgroup drive with runc by editing conf file:
    ```bash
    sudo nano /etc/containerd/config.toml
    ```
    adding this line:
    ```bash
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
    ```

* Restart daemon
    ```bash
    sudo systemctl restart containerd
    ```

### 3. Install Kubernetes Components

* Install required dependencies
    ```bash
    sudo apt-get update \
    && sudo apt-get install -y apt-transport-https curl
    ```

* Add the package repo keys and the repo itself
    ```bash
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    ```

* Update packages and install kubelet
    ```bash
    sudo apt-get update \
    && sudo apt-get install -y -q kubelet kubectl kubeadm
    ```

* Configure cgroup driver for kubelet
    ```bash
    sudo mkdir -p  /etc/systemd/system/kubelet.service.d/

    sudo cat << EOF | sudo tee  /etc/systemd/system/kubelet.service.d/0-containerd.conf
    [Service]
    Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver='systemd'"
    EOF
    ```

* Restart kuebelet
    ```bash
    sudo systemctl daemon-reload \
    && sudo systemctl restart kubelet
    ```

## Do this on kMaster:

* Initialisation with kubeadm
    ```bash
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    ```

* Finish conf setup
    ```bash
    mkdir -p $HOME/.kube \
    && sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config \
    && sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

* Deploy Calico Network
    ```bash
    kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
    ```

* Get cluster join command
    ```bash
    kubeadm token create --print-join-command
    ```

* Get Cluster Nodes
    ```bash
    kubectl get nodes
    ```

## Do this on kWorker:
* Copy and Pate the "Get cluster join command", simliar to:
    ```bash
    kubeadm join 10.0.2.7:6443 --token e2ctwm.barg1e4w35lg4kej --discovery-token-ca-cert-hash sha256:ea81b70854c915a8d965b038f897261affc4889398b46e1cea3b3f1fddd95036
    ```


Done
