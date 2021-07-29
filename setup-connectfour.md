# Deploy and Update a Web-Application

With this guide we deploy a Web-Application for the game Connect Four and update this application by using a different image afterwards. 

For the deployment we need 2 YAML Files. The first one for the deployment of the pods that contains the application itself and the second one for the service that makes sure that
the application can be reached from the outside. For that a Load Balancing Solution like MetalLB should be up and running.

## Deploy a Web-Application

* Create YAML for Deployment by creating the file and pasting the following

    ```bash
    sudo nano connectfour-deployment.yaml
    ```

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: connectfour-deployment
    spec:
      replicas: 2 #numbers of pods 
      selector:
        matchLabels:
          app: connectfour
      template:
        metadata:
          labels:
            app: connectfour
        spec:
          containers:
          - name: connectfour
            image: wentzien/connectfour #docker image of the application
            ports:
            - containerPort: 80
    ```

* Create YAML for Service by creating the file and pasting the following

    ```bash
    sudo nano connectfour-service.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: connectfour-ip-service
    spec:
      type: LoadBalancer
      selector:
        app: connectfour
      ports:
        - port: 80
          targetPort: 80
    ```

* Now both files have to be applied to deploy the application

    ```bash
    kubectl apply -f connectfour-deployment.yaml
    kubectl apply -f connectfour-service.yaml
    ```

* After a few seconds the pods are created and ready. To access the application you need the External IP of the service provided by the Load Balancer. 

    ```bash
    kubectl get services
    ```

## Deploy a Web-Application

Now the appliction is up and running on two pods. In the context of large business applications there are usallay more then 2 pods. The update all pods with a single step we can
simply edit the deployment we created before.

* Get the name of the deployment and edit it

    ```bash
    kubectl get deployments
    kubectl edit deployment connectfour-deployment
    ```

* Edit the image to the new version (@Dennis hier bitte Image anpassen)

* Now the old pods are terminating while the new pods with the new image are starting. This can be seen be the following command. The application is still accessible with the External IP
the Service.

    ```bash
    kubectl get pods
    ```