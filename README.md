# Kubernetes Practice Guide (Minikube + kubectl + Ingress + AWS ECR + Helm)

1️⃣ Install Minikube

    ```bash
    brew install minikube

2️⃣ Create and Start the Cluster

    minikube start --driver=docker
    minikube status
    kubectl get nodes

3️⃣ Main kubectl Commands
Create deployments

    kubectl create deployment nginx-depl --image=nginx
    kubectl create deployment mongo-deployment --image=mongo

View resources

    kubectl get deployments
    kubectl get pods
    kubectl get replicaset
    kubectl get all

Edit deployment

    kubectl edit deployment nginx-depl

View logs

    kubectl logs <pod-name>

Exec into container

    kubectl exec -it <pod-name> -- /bin/bash

Example:

    kubectl exec -it mongo-deployment-xxxx -- /bin/bash

4️⃣ MongoDB Demo (Secrets + ConfigMap + Deployments)

Apply Secret

    kubectl apply -f mongo-secret.yaml
    kubectl get secrets
    Apply MongoDB deployment
    kubectl apply -f mongo.yaml
    kubectl get all | grep mongodb

Apply ConfigMap + Mongo Express

    kubectl apply -f mongo-configmap.yaml
    kubectl apply -f mongo-express.yaml

5️⃣ Create Components in a Namespace
Apply resource to a specific namespace

    kubectl apply -f mongo-configmap.yaml -n my-namespace

6️⃣ Install kubectx (Optional)  

    brew install kubectx

7️⃣ Enable Ingress in MinikubeEnable ingress addon

    minikube addons enable ingress

Open dashboard

    minikube dashboard
    
Check dashboard namespace resources

    kubectl get ns
    kubectl get all -n kubernetes-dashboard

Apply dashboard ingress

    kubectl apply -f dashboard-ingress.yaml
    kubectl get ingress -n kubernetes-dashboard
    kubectl describe ingress dashboard-ingress -n kubernetes-dashboard
    
Add host mapping

    sudo vim /etc/hosts
Add:

    127.0.0.1  dashboard.com
    Start tunnel (keep terminal open)
    minikube tunnel
    Open in browser:

    http://dashboard.com
8️⃣ Mosquitto Exercise (ConfigMap + Secret Volumes)
1) Apply ConfigMap + Secret into the SAME namespace as the deployment
 
    kubectl apply -f config-file.yaml -n my-namespace
    kubectl apply -f secret-file.yaml -n my-namespace
   
2) Verify they exist
 
    kubectl get configmap -n my-namespace | grep mosquitto
    kubectl get secret -n my-namespace | grep mosquitto
   
Expected:

    mosquitto-config-file

    mosquitto-secret-file

3) Restart deployment (or delete pod)
 
    kubectl rollout restart deploy/mosquitto -n my-namespace
### OR
    kubectl delete pod <mosquitto-pod-name> -n my-namespace
    
4) Watch pod status
 
    kubectl get pod -n my-namespace -w
   
Important: If the Deployment mounts /mosquitto/config from a ConfigMap and the ConfigMap is missing/empty, it replaces the folder in the container and Mosquitto may fail to start. For this exercise, the config must come from config-file.yaml.

9️⃣ AWS ECR: Docker Login + imagePullSecret

Login to AWS ECR (run on your machine)

    aws ecr get-login-password --region eu-north-1 \
    | docker login --username AWS --password-stdin 788577008603.dkr.ecr.eu-north-1.amazonaws.com
    
Create Kubernetes Docker registry secret (recommended)

    kubectl create secret docker-registry my-registry-key \
      --docker-server=788577008603.dkr.ecr.eu-north-1.amazonaws.com \
      --docker-username=AWS \
      --docker-password="$(aws ecr get-login-password --region eu-north-1)" \
      -n my-namespace
      
Use imagePullSecrets in Deployment

        spec:
          template:
            spec:
              imagePullSecrets:
                - name: my-registry-key
                
🔟 Helm Basics

Install Helm

     brew install helm
Create a chart

    helm create microservice
    
Render templates (no install)

    helm template emailservice microservice -f email-service-values.yaml
    
Why use helm template:

check templates are valid
catch indentation mistakes
debug values

see final YAML output

Lint chart

    helm lint microservice -f email-service-values.yaml
    
Install chart (correct syntax)

    helm install emailservice microservice -f email-service-values.yaml -n my-namespace

Note: The chart path must be a directory (e.g., microservice).
Typo example to avoid: micriservice (wrong).

Recommended for iteration (install or upgrade)

    helm upgrade --install emailservice microservice -f email-service-values.yaml -n my-namespace

1️⃣1️⃣ Debugging TipsPod status

    kubectl get pods -n my-namespace
    
Describe pod (check Events at the bottom)

    kubectl describe pod <pod-name> -n my-namespace
    
Logs (previous logs help with crash loops)

    kubectl logs <pod-name> -n my-namespace --tail=200
    kubectl logs <pod-name> -n my-namespace --previous --tail=200
