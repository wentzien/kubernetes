# Setup Ingress-Nginx

* Bare-metal installtion:
    [official installtion guide](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/baremetal/deploy.yaml
    ```
* Verify installation
    ```bash
    kubectl get pods -n ingress-nginx \
    -l app.kubernetes.io/name=ingress-nginx --watch
    ```