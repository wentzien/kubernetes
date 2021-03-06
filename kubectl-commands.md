# Useful kubectl commands

## Generally

* cluster info
    ```bash
    kubectl cluter-info
    ```

## Listing resources

* pods
    ```bash
    kubectl get pods #list of all pods

    kubectl get pods -o wide #detailed pods infos incl. node name
    kubectl get pods --field-selector=spec.nodeName=server-name #list of all pods of a specific node server
    ```
* nodes
    ```bash
    kubectl get nodes
    ```

## Get detailed info about an object
    ```bash
    kubectl describe object-type object-name

    # get detailed info of an oject type
    kubectl describe <object-type>
    ```

## Resource management

info: create (imperative management) vs apply (declarative management)
* <span style="color:#009eff">kubectl create</span> will throw an error if resource already exists, <span style="color:#009eff">kubectl apply</span> won't
* <span style="color:#009eff">kubectl create</span> specifically says "do exactly these steps to arrive at this container setup"
* whereas <span style="color:#009eff">kubectl apply</span> says "do whatever is necessary (create, update, ect.) to make it look like this"

### Creating
* run a new pod quickly
    ```bash
    kubectl run pod-name --image=<image-name> --port=80
    ```
* create a resource from a JSON/YAML file
    ```bash
    kubectl create -f file-name
    ```

### Applying and Updating
* create new service, replication controller
    ```bash
    kubectl apply -f <config-file-name>
    kubectl apply -f <directory-name>
    ```
* update by editing it in a text editor (combi of get and apply)
    ```bash
    kubectl edit <type>/<name>
    # example for editing a service:
    kubectl edit svc/<service-name>
    ```
* re-pulling/updating a deployment's image (workaround):
    ```bash
    kubectl set image <object-type>/<object-name> <container-name> = <new-image-name>
    # e.g.
    kubectl set image deployment/client-deployment client=stephengrider/multi-client:v5
    ```
* 
    ```bash
    
    ```

### Deleting
* 
    ```bash
    kubectl delete -f <config-file-name>
    ```
* 
    ```bash
    
    ```

## Abbreviations for Resource Types

Shortcut | Resource type
---- | ----
csr | certificatesigningrequests
cs | componentstatuses
cm | configmaps
ds | daemonsets
deploy | deployments
ep | endpoints
ev | events
hpa | horizontalpodautoscalers
ing | ingresses
limits | limitranges
ns | namespaces
no | nodes
pvc | persistentvolumeclaims
pv | persistentvolumes
po | pods
pdb | poddisruptionbudgets
psp | podsecuritypolicies
rs | replicasets
rc | replicationcontrollers
quota | resourcequotas
sa | serviceaccounts
svc | services