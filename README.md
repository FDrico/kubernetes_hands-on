# Configuration files


Databases --> Statefulset
Stateless apps --> Deployment.

1. `ConfigMap` with MongoDB endpoint
    - With key-value pairs as the main config.
2. `Secret` with MongoDB user and password
    - Values stored with base64 encoding. `echo -n my_secret | base64`.
3. `Config files` for Deployments
    - Template holds the configuration for the pods, as a deployment manages pods.
        - The template has its own metadata and spec section, with the images to be ran by the containers.
        - container images can be found at docker-hub.
    - Components can have key-value pairs.
        - labels = identifiers for the user.
        - Each pod/replica have a unique name but they all share the same label.
        - matchLabels maps the labels of the pods with a certain deployment.
    - Envvars can be set using the `env` key, and secrets can be read from files.
4. `Config files` for Services
    - We can write them on the same yaml file as the Deployments.
    - Services have a selector because that's were it forwards requests to (should use the same label selector as define on the containers).
    - Port can be anything, targetPort is the one that belongs to the service. Should be the same as the container port.

    ```mermaid
    graph LR;
    Request --port8080-->MongoService--targetPort27017-->Pod
    ```
    - NodePort: External service type. Requires us to define a nodePort, between 30000 and 32767.


# Start minikube

```
minikube start --driver docker
minikube stop
```
# Check status
```
minikube status
kubectl get node # check clusters running on the local machine
```

# Create components in Kubernetes
With minikube cluster running
```
kubectl get pod
# Load configs
kubectl apply -f mongo-config.yml
kubectl apply -f mongo-secret.yml
# Create the database and webapp
kubectl apply -f mongo.yml
kubectl apply -f webapp.yml
```
```
kubectl get all # Get all components
NAME                                     READY   STATUS             RESTARTS      AGE
pod/mongo-deployment-5f6686d565-j6pr2    0/1     Error              3 (27s ago)   59s
pod/mongo-deployment-5f6686d565-kgrms    0/1     CrashLoopBackOff   2 (26s ago)   59s
pod/mongo-deployment-5f6686d565-pwv7g    0/1     CrashLoopBackOff   2 (17s ago)   59s
pod/webapp-deployment-758fd7bc4d-gdr4r   1/1     Running            0             22s
pod/webapp-deployment-758fd7bc4d-j6dcd   1/1     Running            0             22s
pod/webapp-deployment-758fd7bc4d-j7lql   1/1     Running            0             22s

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          12m
service/mongo-service    ClusterIP   10.107.73.114    <none>        80/TCP           59s
service/webapp-service   NodePort    10.102.129.208   <none>        3000:30100/TCP   22s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo-deployment    0/3     3            0           59s
deployment.apps/webapp-deployment   3/3     3            3           22s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-deployment-5f6686d565    3         3         0       59s
replicaset.apps/webapp-deployment-758fd7bc4d   3         3         3       22s
```

```
kubectl get configmap        
    NAME               DATA   AGE
    kube-root-ca.crt   1      12m
    mongo.config       1      2m8s

kubectl get pod      
    NAME                                 READY   STATUS             RESTARTS      AGE
    mongo-deployment-5f6686d565-j6pr2    0/1     CrashLoopBackOff   3 (52s ago)   110s
    mongo-deployment-5f6686d565-kgrms    0/1     Error              4 (50s ago)   110s
    mongo-deployment-5f6686d565-pwv7g    0/1     CrashLoopBackOff   3 (39s ago)   110s
    webapp-deployment-758fd7bc4d-gdr4r   1/1     Running            0             73s
    webapp-deployment-758fd7bc4d-j6dcd   1/1     Running            0             73s
    webapp-deployment-758fd7bc4d-j7lql   1/1     Running            0             73s
```

```
kubectl describe service webapp-service
Name:                     webapp-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=webapp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.102.129.208
IPs:                      10.102.129.208
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  30100/TCP
Endpoints:                10.244.0.6:3000,10.244.0.7:3000,10.244.0.8:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```
kubectl describe pod webapp-deployment-758fd7bc4d-j7lql # details about pod, labels, configurations, etc.
```

```
kubectl logs mongo-deployment-5f6686d565-j6pr2
```

```
kubectl get svc
```

External (nodeport) services are available though the host ip, which can be obtained by:

```
minikube ip
OR
kubectl get node -o wide
```
