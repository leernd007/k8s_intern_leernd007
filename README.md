DO_2

1. Create AKS cluster and pull it using cli
2. Install HELM 
3. Install nginx ingress controller
    ```
    helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace
    ```
4. Wait while load balancer EXTERNAL IP will be loaded and remember it. In my case I got **20.215.89.28**
5. Pick up kube app
    ```
    kubectl create -f .
    ```
6. Specify DNS name
   ```
   sudo nano /etc/hosts
   ```
   ```
   20.215.89.28 tst.devops.com
   ```
7. In browser open **20.215.89.28** or **http://tst.devops.com** and you will see new created app