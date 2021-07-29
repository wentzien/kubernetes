# Monero Mining with XMRIG

Link to GitHub Repo [GitHub:wentzien/xmrig](https://github.com/wentzien/xmrig) and Docker Hub image [Docker:wentzien/xmrig](https://hub.docker.com/repository/docker/wentzien/xmrig)

## Instructions
You can either use the image with Docker or on Kubernetes.

### using Docker
* install on the host the required packages for max hashrate:
    ```bash
    sudo apt -y install kmod msr-tools
    ```
* run container
    with default settings:
    ```docker
    docker run --name xmrig --privileged --cap-add ALL -v /lib/modules:/lib/modules wentzien/xmrig:1.0.4
    ```
    with custom settings:
    ```docker
    docker run --name xmrig \
        --env pool=<host-and-port-of-mining-pool> \
        --env wallet_address=<your-wallet-address> \
        --env rig_id=<custom-rig-id-name> \
        --env donate_level=<donation-to-pool-in-percent> \
        --privileged \
        --cap-add ALL \
        -v /lib/modules:/lib/modules \
        wentzien/xmrig:1.0.4

    #eg
    sudo docker run --name xmrig \
        --env pool=de.minexmr.com:443 \
        --env wallet_address=4AePnmc5NxmNWArsL3tLgF8hyMeqnzzNMLTY4tEc5DSdVKCEijp4m7sckeUFU5ACChgVhoFHHasi2DFDFGp1METwNPDMbDs \
        --env rig_id="mein mining rig" \
        --env donate_level=1 \
        --privileged \
        --cap-add ALL \
        -v /lib/modules:/lib/modules \
        wentzien/xmrig:1.0.4
    ```


### using Kubernetes
* use kube-xmrig-deployment.yaml
* install on every node the required packages for max hashrate:
    ```bash
    sudo apt -y install kmod msr-tools
    ```
* adjust env variable values in kube-xmrig-deployment.yaml
* deploy in namespace "xmrig":
    ```bash
    kubectl apply -f kube-xmrig-deployment.yaml -n xmrig
    ```
* show all pods:
    ```bash
    kubectl get pod -n xmrig
    ```
* show logs of a specific pod:
    ```bash
    kubectl logs <name-of-pod>

    # eg
    # kubectl logs xmrig-deployment-7b675997-9x4n9
    ```

Enjoy mining! :)
