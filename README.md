DO_2

1. Create AKS cluster and pull it using cli
2. Create ACR secret
   ```
   kubectl create secret docker-registry acr-secret \
   --namespace app-tst \
   --docker-server=<...> \
   --docker-username=<...> \
   --docker-password=<...>
   ```
3. Install HELM 
4. Install nginx ingress controller
    ```
    helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace
    ```
5. Wait while load balancer EXTERNAL IP will be loaded and remember it. In my case I got **20.215.89.28**
6. Create Namespace **app-tst**
   ```
   kubectl create ns app-tst
   ```
7. Pick up kube app
    ```
    kubectl create -f .
    ```
8. Specify DNS name
   ```
   sudo nano /etc/hosts
   ```
   ```
   20.215.89.28 tst.devops.com
   ```
9. In browser open **20.215.89.28** or **http://tst.devops.com** and you will see new created app

DO_3

1. Create one more nginx controller with another class
   ```
   helm upgrade --install ingress-nginx-prod ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx-prod --set controller.ingressClassResource.name=nginx-prod \
   --set controller.ingressClassResource.controllerValue="k8s.io/ingress-nginx-prod" \
   --set controller.ingressClassResource.enabled=true \
   --set controller.IngressClassByName=true \
   --set controller.ingressClass=nginx-prod
   ```
2. Create one more ACR secret with different namespace
   ```
   kubectl create secret docker-registry acr-secret \
   --namespace app-pro \
   --docker-server=<...> \
   --docker-username=<...> \
   --docker-password=<...>
   ```
3. Create Namespace **app-pro**
   ```
   kubectl create ns app-pro
   ```
4. Pick up app1
   ```
   kubectl create -f app/ -n app-tst
   ```
5. Pick up app2
   ```
   kubectl create -f app/ -n app-pro
   ```
6. Pick up Ingress
   ```
   kubectl create -f ingress.yaml
   ```
7. Get EXTERNAL IP. In my case I got **20.215.180.214**
8. Specify DNS name
   ```
   sudo nano /etc/hosts
   ```
   ```
   20.215.180.214 pro.devops.com
   ```
9. In browser open **20.215.180.214** or **http://pro.devops.com** and you will see new created app
10. Check connectivity between different namespaces. 
    <br>Let's connect to **frontend** pod with **app-pro** namespace and check connection to **backend** with **app-tst** namespace
   ```
   kubectl exec frontend-deployment-5ccf4844d-pl9lf  -it sh  -n app-pro
   curl 10.0.169.93:8000
   ```
   where **10.0.169.93** is Cluster IP to **backend** with **app-tst** namespace. **8000** port for backend resolution.<br>
   As result I got
   **{"detail":"Not Found"}**, so connection exists
11. Apply network policy
   ```
   kubectl apply -f deny-from-other-namespaces.yaml
   ```
12. Try again
   ```
   kubectl exec frontend-deployment-5ccf4844d-pl9lf  -it sh  -n app-pro
   curl 10.0.169.93:8000
   ```
13. As result, I got the error
   ````
   ```
   # curl 10.0.169.93:8000
   curl: (28) Failed to connect to 10.0.169.93 port 8000 after 129459 ms: Operation timed out
   ```
   ````

    