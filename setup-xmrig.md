# Monero Mining with XMRIG

Link to GitHub Repo [GitHub:wentzien/xmrig](https://github.com/wentzien/xmrig) and Docker Hub image [Docker:wentzien/xmrig](https://hub.docker.com/repository/docker/wentzien/xmrig)

## Deploy on Kubernetes

### Requirements
* at least 2.5 Gi Memory
* Wallet for cryptocurrencies is needed
* recommended to run one pod on one node for max efficiency

### Instructions
* create kube-xmrig-deployment.yaml
    ```bash
    vim kube-xmrig-deployment.yaml
    ```
* add content to yaml and adjust env variable values:
    attention: for the maximum hashrate, the pod needs access to the msr kernel of the host (node), so / lib / modules is mounted from the host to the pod (this is of course a security risk)
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: xmrig-deployment
    labels:
        app: xmrig
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: xmrig
    template:
        metadata:
        labels:
            app: xmrig
        spec:
        containers:
        - name: xmrig
            image: wentzien/xmrig:1.0.1
            env:
            - name: pool
            value: de.minexmr.com:443
            - name: wallet_address
            value: 4AePnmc5NxmNWArsL3tLgF8hyMeqnzzNMLTY4tEc5DSdVKCEijp4m7sckeUFU5ACChgVhoFHHasi2DFDFGp1METwNPDMbDs
            - name: rig_id
            value: "k8xmrigmining"
            - name: donate_level
            value: "1"
            resources:
            requests:
                cpu: "2000m"
                memory: 2.5Gi
            securityContext:
            privileged: true
            volumeMounts:
            - mountPath: "/lib/modules"
            name: moduleslib
        volumes:
        - name: moduleslib
            hostPath:
            path: "/lib/modules"
    ```
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
