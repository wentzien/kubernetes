# Problems and how to solve it

* The connection to the server host-ip:6443 was refused - did you specify the right host or port?
    Try to disable swap again:
    ```bash
    sudo -i
    swapoff -a; sed -i '/swap/d' /etc/fstab
    ```
* Nodes are in status NotReady
    ```bash
    kubectl describe nodes
    ```
    if: "container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
    -> forgot to deploy calico network
    ```bash
    kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
    ```
