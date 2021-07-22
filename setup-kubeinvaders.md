# Setup KubeInvaders

## What is KubeInvaders?
KubeInvaders is a gamified Chaos-Engineering Tool to test the availabilty of your cluster. With KubeInvaders you can kill pods and nodes which are 
displayed as Aliens from the game "Space Invaders". KubeInvaders is availble on Github: https://github.com/lucky-sideburn/KubeInvaders. <br> <br>
For our use-case we forged the official Github repo because we have to sligthly modify some parameters to be able to run KubeInvaders with our
Bare-Metal Kubernetes. For that reason we also upload our own version of an docker image. 

## Prerequisites
* Helm should be installed for easy installation
* Metal-LB should be installed to be able to specify service type LoadBalancer for the KubeInvaders Installation

## Install KubeInvaders

* Create namespace where KubeInvaders will be installed (this is not the namespace that will be targeted by KubeInvaders):
    ```bash
    kubectl create namespace kubeinvaders
    ```
* (Optional) namespaces that will be targeted by KubeInvaders:
    ```bash
    kubectl create namespace kube-target
    kubectl create namespace kube-target-more-apps
    ```

* Clone our forged Github repo
    ```bash
    git clone https://github.com/wentzien/KubeInvaders && cd KubeInvaders
    ```
   
* Install KubeInvaders with Helm and dummy route host. Here it is possible the targeted namespaces. For our use-case we use two namespaces that only run applications for KubeInvaders.
    ```bash
    helm install kubeinvaders ./helm-charts/kubeinvaders \
    --version 0.1.1 \
    --namespace kubeinvaders \
    --set target_namespace="kube-target\,kube-target-more-apps" \
    --set route_host=dummy
    ```
* Get External-IP of the Kubernetes Service
    ```bash
    kubectl get services -n kubeinvaders
    ```

* Upgrade KubeInvaders with the correct route host
    ```bash
    helm upgrade kubeinvaders ./helm-charts/kubeinvaders \
    --version 0.1.2 \
    --namespace kubeinvaders \
    --set target_namespace="kube-target\,kube-target-more-apps" \
    --set route_host=''YOUR_EXTERNAL_IP_OF_KUBEINVADERS_SERVICE''
    ```

* Create Pods in the targeted namespaces
    ```bash
    kubectl create deployment nginx --image=nginx --namespace kube-target
    kubectl create deployment ghost --image=ghost:3.11.0-alpine --namespace kube-target-more-apps
    
    kubectl scale deployment/nginx --replicas=8 --namespace kube-target
    kubectl scale deployment/ghost --replicas=4 --namespace kube-target-more-apps
    ```

* Label Deployments and Pods to better monitor their status
    ```bash
    kubectl label deployment,pod app-purpose=chaos -l app=nginx --namespace kube-target
    kubectl label deployment,pod app-purpose=chaos -l app=ghost --namespace kube-target-more-apps
    ```

* The following can be used to monitor the pods when KubeInvaders is running to see which pods are killed and replaced
    ```bash
    watch kubectl get deployments,pods --all-namespaces -l app-purpose=chaos
    ```

KubeInvaders is now accessible via to IP Address of the KubeInvaders Service provided by the MetalLB.
