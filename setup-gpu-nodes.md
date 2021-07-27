# GPU Support Setup


* optional for singlenode: allow the master to execute nodes as well
    ```bash
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```
* Prerequisites
    * container engine (eg. containerd)
    * nouveau driver for NVIDIA GPUs must be blacklisted (on HWE kerne -> Ubuntu 18.04 LTS or 20.04 LTS)
        open with editor:
        ```bash
        sudo vim /etc/modprobe.d/blacklist-nouveau.conf
        ```
        add contents:
        ```
        blacklist nouveau
        ptions nouveau modeset=0
        ```
        regenerate kernel
        ```bash
        sudo update-initramfs -u
        ```
    * look if Node Feature Discovery (NFD) is already deployed... if so, configure the Operator to not install NFD

* Install helm, if not already installed:
    ```bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
        && chmod 700 get_helm.sh \
        && ./get_helm.sh
    ```

* add NVIDIA Helm repository:
    ```bash
    helm repo add nvidia https://nvidia.github.io/gpu-operator \
        && helm repo update
    ```

* Common Deployment Scenarios (on Bare-metal with containerd)
    * default:
        ```bash
        helm install --wait --generate-name \
            nvidia/gpu-operator
            --set operator.defaultRuntime=containerd
        ```
    * if NVIDIA drivers are pre-installed:
        ```bash
        helm install --wait --generate-name \
            nvidia/gpu-operator \
            --set operator.defaultRuntime=containerd
            --set driver.enabled=false
        ```