# Install JupyterHub

## Setup NFS 
* Install NFS Server Setup on the PC that should be used as storage pc (in our case Worker 1)
    ```bash
    sudo apt install nfs-kernel-server
    ```

* @all other PCs in Network: Install NFS Client Setup
    ```bash
    sudo apt install nfs-common
    ```

* Next you have to setup a folder on the NFS server
    ```bash
    sudo mkdir /srv/nfs/kubedata -p
    sudo chown nobody: /srv/nfs/kubedata
    ```

* Now this folder has to be exposed by adding the following to the /etc/exports file
    ```
    /srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
    ```

* After restarting the NFS Server the folder will be available
    ```bash
    sudo systemctl enable --now nfs-server
    sudo exportfs -rav
    ```

* Now the NFS Server is up and running and we can go back to the Master Node to use the storage in our Kubernetes System. For that we use Helm to install an NFS-Provisioner-Tool for Kubernetes (https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/). First we add the Repo to our Helm installation and then install the NFS with some extra parameters. We have to define the IP and the Path of the NFS Server. 
    ```bash
    helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=10.42.0.8 \
    --set nfs.path=/srv/nfs/kubedata \
    --set parameters.archiveOnDelete=false
    ```

* With the following command we first show the name of the Storage Class we created. To more easliy handle the upcoming sets we set this Storage class as default.
    ```bash
    kubectl get sc
    kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

## Deploy JupyterHub

* Generate random hex string to use as a security token
    ```bash
    openssl rand -hex 32
    ```
  
* Create a config.yaml with service type LoadBalancer and the information from the NFS Server for data storage. The parameter capacity defines the amount of storage every user of JupyterHub can use. 

    ```yaml
    proxy:
        secretToken: "<random-hex-token>"
        service:
            type: LoadBalancer

    hub:
        db:
            type: sqlite-pvc
            storage: 1Gi
    
    singleuser:
        storage:
            type: dynamic
            capacity: 5Gi
    ```

* Add JupyterHub Helm chart repository
    ```bash
    helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
    helm repo update
    ```
* Install the chart configured by the config.yaml
    ```bash
    RELEASE=jhub
    NAMESPACE=jhub

    helm upgrade --cleanup-on-fail \
    --install $RELEASE jupyterhub/jupyterhub \
    --namespace $NAMESPACE \
    --create-namespace \
    --version=0.11.1 \
    --values config.yaml
    ```
* Inspect the JupyterHub Installtion
    ```bash
    kubectl get all --namespace jhub
    ```

* Get the IP Address of JupyterHub provided by the LoadBalancer
    ```bash
    kubectl get all --namespace jhub
    ```


* Delete the installation:
    ```bash
    helm delete jhub --namespace jhub
    ```
