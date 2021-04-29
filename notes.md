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
            * Good for development, rarely used in production

        * Service Object
            * Purpose: Sets up networking in a K8s Cluster
                * type: ClusterIP, NodePort (exposes a container to the outside world - dev only), LoadBalancer, Ingress
                * selector: Im going to use my selector "component: web" to find every other object with a label of "component: web" and expose its port 3000 to the outside world
                * ports: (array)
                    * port: another container/pod can get access to this pod 
                    * targerPort: identical to the containerPort
                    * nodePort: to access the pod e.g. via browser (if not specified port number ist random between 30000-32767)

        * Deployment:
            * Purpose: Maintains a set of identical pods, ensuring that they have the correct config and that the right number exists
            * Good for development and production
            * Handels all created pods with the selector field  

* Triggering Deployment Updates
    * Problem: Force pods to re-pull an image without changing the image tag
    * Solution/Workaround: Use of an imperative command

* create (imperative management) vs apply (declarative management)
    * <span style="color:#009eff">kubectl create</span> will throw an error if resource already exists, <span style="color:#009eff">kubectl apply</span> won't
    * <span style="color:#009eff">kubectl create</span> specifically says "do exactly these steps to arrive at this container setup"
    * whereas <span style="color:#009eff">kubectl apply</span> says "do whatever is necessary (create, update, ect.) to make it look like this"

* Store data of a database with a persistent volume. Falls an old pod away, a new pod can connect to this persitent volume and no data is lost.