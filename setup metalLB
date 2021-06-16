# Setup MetalLB

## Why MetalLB is necessary?
* Kubernetes in a Cloud Environment like AWS usally can use pre-defined LoadBalancers to assign external IPs to your services so you can access your applications
* A bare-metal Kubernetes Cluster (usage of local pcs/servers) does not have pre-defined LoadBalancers
* MetalLB is a LoadBalancer implementation for bare-metal Kubernetes

## Prerequisites
* you should have 10-20 unused IPs in your network that you can assign to MetalLB
* for further requirements see official documentation: https://metallb.universe.tf/#requirements

## Do this on kMaster:

* Installation MetalLB by Manifest:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
    ```

* Configure MetalLB by creating a yaml-file
    ```bash
    sudo nano /tmp/metallb.yaml
    ```
   
* Paste the following into the yaml-file. The last line defines the free IP-Adresses for the LoadBalancer
    ```bash
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 10.42.0.100-10.42.0.120
    ```

* Check if LoadBalancer is working by getting information about services. If a exposed service has a external IP in aboves range, the LoadBalancer is working
    ```bash
    kubectl get services
    ```

Done
