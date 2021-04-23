# General Notes

* Docker Compose vs. Kubernetes
    Docker Compose | Kubernetes | Run a container on a local Kubernetes Cluster
    ---- | ---- | ----
    Each entry can optionally get docker-compose to build an image | Kubernetes expects all images to already be built | Make sure the image is hosted on docker hub
    Each entry represents a container we want to create | One config file per object we want to create | Make one config file to create the container
    Each entry defines the networking requirements (ports) | Set up manually all networking | Make one config file to set up networking

* Configuration Files
    * apiVersion: each API version defines a different set of objects, which ca be used
        * v1: componentStatus, configMap, Endpoints, Event, Namespace, Pod
        * apps/v1: ControllerRevision, StatefulSet
    * metadata:
        * name: name the pod which will be created, mostly used for loggin services
        * labels: 
    * kind: Objects serve different purposes - running a container, monitoring a container, setting up networking, etc.; Example Types: StatefulSet, ReplicaController, Pod, Serice
        * Pod Object
            * Purpose: Run a groupe of containers with a very common purpose
            * E.g. Pod with postgres as primary container and logger & backup-manager as support container

        * Service Object
            * Purpose: Sets up networking in a K8s Cluster
                * Types: ClusterIP, NodePort (exposes a container to the outside world - dev only), LoadBalancer, Ingress
                * Selector: Im going to use my selector "component: web" to find every other object with a label of "component: web" and expose its port 3000 to the outside world
