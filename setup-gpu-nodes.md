# GPU Support Setup


* optional for singlenode: allow the master to execute nodes as well
    ```bash
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```
* Prerequisites
    * container engine (eg. containerd)
    * nouveau driver for NVIDIA GPUs must be blacklisted (on HWE kernel -> Ubuntu 18.04 LTS or 20.04 LTS)
        open with editor:
        ```bash
        sudo vim /etc/modprobe.d/blacklist-nouveau.conf
        ```
        add contents:
        ```
        blacklist nouveau
        ptions nouveau modeset=0
        ```
        regenerate kernel
        ```bash
        sudo update-initramfs -u
        ```
    * look if Node Feature Discovery (NFD) is already deployed... if so, configure the Operator to not install NFD

* Install helm, if not already installed:
    ```bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
        && chmod 700 get_helm.sh \
        && ./get_helm.sh
    ```

* add NVIDIA Helm repository:
    ```bash
    helm repo add nvidia https://nvidia.github.io/gpu-operator \
        && helm repo update
    ```

* Common Deployment Scenarios (on Bare-metal with containerd)
    * default:
        ```bash
        helm install --wait --generate-name \
            nvidia/gpu-operator
            --set operator.defaultRuntime=containerd
        ```
    * if NVIDIA drivers are pre-installed:
        ```bash
        helm install --wait --generate-name \
            nvidia/gpu-operator \
            --set operator.defaultRuntime=containerd
            --set driver.enabled=false
        ```

* test GPU setup
    * look at the pods
        ```bash
        kubectl get pods -A
        ```
        similar output to:
        ```bash
        default                  gpu-operator-1627395640-node-feature-discovery-master-75ffcmbd9   1/1     Running     0          68m
        default                  gpu-operator-1627395640-node-feature-discovery-worker-ng8r7       1/1     Running     0          68m
        default                  gpu-operator-6b5666bb8b-qb8zx                                     1/1     Running     0          68m
        gpu-operator-resources   gpu-feature-discovery-vjts9                                       1/1     Running     0          68m
        gpu-operator-resources   nvidia-container-toolkit-daemonset-mw8tr                          1/1     Running     0          68m
        gpu-operator-resources   nvidia-cuda-validator-2jc2r                                       0/1     Completed   0          67m
        gpu-operator-resources   nvidia-dcgm-exporter-sgkhv                                        1/1     Running     0          68m
        gpu-operator-resources   nvidia-device-plugin-daemonset-kgcqh                              1/1     Running     0          68m
        gpu-operator-resources   nvidia-device-plugin-validator-zml4k                              0/1     Completed   0          66m
        gpu-operator-resources   nvidia-operator-validator-d8cgf                                   1/1     Running     0          68m
        kube-system              calico-kube-controllers-7676785684-x9zcw                          1/1     Running     0          104m
        kube-system              calico-node-vssww                                                 1/1     Running     0          104m
        kube-system              coredns-558bd4d5db-8gk5s                                          1/1     Running     0          105m
        kube-system              coredns-558bd4d5db-k7bxm                                          1/1     Running     0          105m
        kube-system              etcd-ubuntu20                                                     1/1     Running     0          105m
        kube-system              kube-apiserver-ubuntu20                                           1/1     Running     0          105m
        kube-system              kube-controller-manager-ubuntu20                                  1/1     Running     0          105m
        kube-system              kube-proxy-ddht2                                                  1/1     Running     0          105m
        kube-system              kube-scheduler-ubuntu20                                           1/1     Running     0          105m
        ```
    * run CUDA VectorAdd
        ```bash
        cat << EOF | kubectl create -f -
        apiVersion: v1
        kind: Pod
        metadata:
        name: cuda-vectoradd
        spec:
        restartPolicy: OnFailure
        containers:
        - name: cuda-vectoradd
            image: "nvidia/samples:vectoradd-cuda11.2.1"
            resources:
            limits:
            nvidia.com/gpu: 1
        EOF
        ```
        get the logs:
        ```bash
        kubectl logs cuda-vectoradd
        ```
        output similar to:
        ```bash
        [Vector addition of 50000 elements]
        Copy input data from the host memory to the CUDA device
        CUDA kernel launch with 196 blocks of 256 threads
        Copy output data from the CUDA device to the host memory
        Test PASSED
        Done
        ```
    * run CUDA FP16 Matrix multiply
        ```bash
        cat << EOF | kubectl create -f -
        apiVersion: v1
        kind: Pod
        metadata:
        name: dcgmproftester
        spec:
        restartPolicy: OnFailure
        containers:
        - name: dcgmproftester11
            image: nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
            args: ["--no-dcgm-validation", "-t 1004", "-d 30"]
            resources:
            limits:
                nvidia.com/gpu: 1
            securityContext:
            capabilities:
                add: ["SYS_ADMIN"]
        EOF
        ```
        get the logs:
        ```bash
        kubectl logs -f dcgmproftester
        ```
        output similar to:
        ```
        Skipping CreateDcgmGroups() since DCGM validation is disabled
        CU_DEVICE_ATTRIBUTE_MAX_THREADS_PER_MULTIPROCESSOR: 2048
        CU_DEVICE_ATTRIBUTE_MULTIPROCESSOR_COUNT: 28
        CU_DEVICE_ATTRIBUTE_MAX_SHARED_MEMORY_PER_MULTIPROCESSOR: 98304
        CU_DEVICE_ATTRIBUTE_COMPUTE_CAPABILITY_MAJOR: 6
        CU_DEVICE_ATTRIBUTE_COMPUTE_CAPABILITY_MINOR: 1
        CU_DEVICE_ATTRIBUTE_GLOBAL_MEMORY_BUS_WIDTH: 352
        CU_DEVICE_ATTRIBUTE_MEMORY_CLOCK_RATE: 5505000
        Max Memory bandwidth: 484440000000 bytes (484.44 GiB)
        CudaInit completed successfully.

        Skipping WatchFields() since DCGM validation is disabled
        TensorEngineActive: generated ???, dcgm 0.000 (183.9 gflops)
        TensorEngineActive: generated ???, dcgm 0.000 (194.4 gflops)
        etc...
        ```
    * run Jupyter Notebook
        ```bash
        deploy example notebook:
        kubectl apply -f https://nvidia.github.io/gpu-operator/notebook-example.yml
        ```
        check if pod started:
        ```bash
        kubectl get pod tf-notebook
        ```
        output similar to:
        ```bash
        NAME          READY   STATUS    RESTARTS   AGE
        tf-notebook   1/1     Running   0          64m
        ```
        get external port:
        ```bash
        kubectl get svc -A
        ```
        output similar to:
        ```bash
        NAMESPACE                NAME                                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  
        default                  tf-notebook                                             NodePort    10.111.138.247   <none>        80:30001/TCP             65m
        ```
        get the token:
        ```bash
        kubectl logs tf-notebook
        ```
        output similar to:
        ```bash
        [I 14:38:24.697 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
        [I 14:38:24.856 NotebookApp] Serving notebooks from local directory: /tf
        [I 14:38:24.856 NotebookApp] Jupyter Notebook 6.3.0 is running at:
        [I 14:38:24.856 NotebookApp] http://tf-notebook:8888/?token=3ba71495882d96f6157dbc5c35b6d8a85bca61d4da6710d8
        [I 14:38:24.856 NotebookApp]  or http://127.0.0.1:8888/?token=3ba71495882d96f6157dbc5c35b6d8a85bca61d4da6710d8
        [I 14:38:24.856 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
        [C 14:38:24.860 NotebookApp] 
            
            To access the notebook, open this file in a browser:
                file:///root/.local/share/jupyter/runtime/nbserver-1-open.html
            Or copy and paste one of these URLs:
                http://tf-notebook:8888/?token=3ba71495882d96f6157dbc5c35b6d8a85bca61d4da6710d8
            or http://127.0.0.1:8888/?token=3ba71495882d96f6157dbc5c35b6d8a85bca61d4da6710d8
        ```



