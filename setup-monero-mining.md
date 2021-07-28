# Setup Monero Mining with XMRIG

## Prerequisites
* Nodes should have at least 8 GB of RAM
* Wallet for cryptocurrencies is needed
* Helm should be installed for easy installation

## Setup XMRIG
* Add XMRIG Helm Repo
    ```bash
    helm repo add brannon https://helm.brannon.online
    ```

* Generate a yaml file with some parameters. Here we use to pool from Minexmr. The parameter Email is the address of the wallet. The parameter password is not used and filled with a dummy value.
    ```bash
    echo '
    pool: de.minexmr.com:4444
    email: <<YOUR_WALLET_ADDRESS>>
    password: xx
    ' > xmrig-value-overrides.yaml
    ```

* Now XMRIG is installed with Helm by using the values we defined before. 
    ```
    kubectl create namespace monero
    helm upgrade --install xmrig brannon/xmrig --values xmrig-value-overrides.yaml -n monero
    ```

* Now XMRIG is running with 1 Pod, but has to be further modified by editing the deployment.
    ```bash
    kubectl edit deployment xmrig -n monero
    # set replicas to a higher value
    # delete all the parameters for password around line 55
    ```

* Now multiple pods are up and running. With following command it is possible to check the the status of a pod.
    ```bash
    kubectl logs --namespace monero <<POD_NAME>>
    ```
