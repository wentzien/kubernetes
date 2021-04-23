# Useful kubectl commands

## Listing resources

* pods
    ```bash
    kubectl get pods #list of all pods
    kubectl get po # list of all pods

    kubectl get po -o wide #detailed pods infos incl. node name
    kubectl get po --field-selector=spec.nodeName=server-name #list of all pods of a specific node server
    ```
* nodes
    ```bash
    kubectl get nodes
    kubectl get no
    ```

## Resource management

info: create (imperative management) vs apply (declarative management)
* <span style="color:#009eff">kubectl create</span> will throw an error if resource already exists, <span style="color:#009eff">kubectl apply</span> won't
* <span style="color:#009eff">kubectl create</span> specifically says "create this thing"
* whereas <span style="color:#009eff">kubectl apply</span> says "do whatever is necessary (create, update, ect.) to make it look like this"

### Creating
* run a new pod quickly
    ```bash
    kubectl run pod-name --image=image-name --port=80
    ```
* create a resource from a JSON/YAML file
    ```bash
    kubectl create -f file-name
    ```

### Applying
* 
    ```bash
    
    ```
* 
    ```bash
    
    ```

### Updating
* 
    ```bash
    
    ```
* 
    ```bash
    
    ```

### Deleting
* 
    ```bash
    
    ```
* 
    ```bash
    
    ```