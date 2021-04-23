# Problems and how to solve it

* The connection to the server host-ip:6443 was refused - did you specify the right host or port?
    Try to disable swap again:
    ```bash
    sudo -i
    swapoff -a; sed -i '/swap/d' /etc/fstab
    ```