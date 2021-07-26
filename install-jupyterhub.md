# Install JupyterHub

* Generate random hex string to use as a security token
    ```bash
    openssl rand -hex 32
    ```
* Create a config.yaml
    Simple settings for testing and quick deploy (using NodePort instead of a LoadBalancer)
    ```yaml
    proxy:
        secretToken: "<random-hex-token>"
        service:
            type: NodePort
            nodePorts:
                http: 30080
                https: 30443

    hub:
        db:
            type: sqlite-memory

    singleuser:
        storage:
            type: sqlite-memory
    ```

    ```yaml
    proxy:
    secretToken: 84283cbb9a7c882cc4b0bf958a54c6317a12a77340eb6a4897e5902da54d54f7

    hub:
        cookieSecret: 89e06e14dd69175b4353cf48f6156bd48e8e5539aee5b62c383ec3732396b36d
        db:
            type: mysql
            url: mysql+pymysql://<dbuser>:<password>@<host/ip>:3306/<database>

    singleuser:
    type:
            storage:
                        type: sqlite-memory
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
    --version=0.10.6 \
    --values config.yaml
    ```
* Inspect the JupyterHub Installtion
    ```bash
    kubectl get all --namespace jhub
    ```

* Delete the installation:
    ```bash
    helm delete jhub --namespace jhub
    ```
