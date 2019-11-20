# Kubernetes example

This example is based on ["Mumshad Mannambeth" course "kubernetes for the absolute beginners"](https://www.udemy.com/share/1013LOBkcec1pTQXw=/)

The used docker images are retrieved from the [docker voting app samples](https://hub.docker.com/search?q=dockersample&type=image)

The example is run in GCP kubernetes engine

## Create the voting app using pods and services

### Steps

1. Deploy PODS
2. Create ClusterIP Services (for internal communication in the cluster)
3. Create Loadbalancer Service (to expose the front application to external users)

![Application architecture](/resources/VotingApp.png)

```
kubectl create -f voting-app-pod.yml // to create a pod from a yml file
kubectl get pods // to check that your pod is welle created
kubectl create -f voting-app-service.yml // to create a service from a yml file. the service type is LoadBalncer, it will only work on public cloud plateforms as GCP
kubectl get services //ti check that your service is well created

The output will be something as follow :

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.83.0.1 <none> 443/TCP 19h
voting-service LoadBalancer 10.83.11.96 35.238.94.250 80:32404/TCP 47s
```

To check that the voting application is correctly running, go to http://{voting-service.EXTERNAL-IP} ==> The voting app must be displayed

```
kubectl create -f redis-pod.yml
kubectl get pods
kubectl create -f redis-service.yml
kubectl get services

The output will be something as follow :

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.83.0.1 <none> 443/TCP 20h
redis ClusterIP 10.83.5.24 <none> 6379/TCP 23s
voting-service LoadBalancer 10.83.11.96 35.238.94.250 80:32404/TCP 7m24s
```

```
kubectl create -f postgres-pod.yml
kubectl get pods
kubectl create -f postgres-service.yml
kubectl get services

The output will be something as follow :

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
db ClusterIP 10.83.12.76 <none> 5432/TCP 11s
kubernetes ClusterIP 10.83.0.1 <none> 443/TCP 20h
redis ClusterIP 10.83.5.24 <none> 6379/TCP 4m42s
voting-service LoadBalancer 10.83.11.96 35.238.94.250 80:32404/TCP 11m
```

```
kubectl create -f worker-app-pod.yml
kubectl get pods
```

```
kubectl create -f result-app-pod.yml
kubectl get pods
kubectl create -f result-app-service.yml
kubectl get services

The output will be something as follow :

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
db ClusterIP 10.83.12.76 <none> 5432/TCP 4m16s
kubernetes ClusterIP 10.83.0.1 <none> 443/TCP 20h
redis ClusterIP 10.83.5.24 <none> 6379/TCP 8m47s
result-service LoadBalancer 10.83.10.90 <pending> 80:31100/TCP 25s
voting-service LoadBalancer 10.83.11.96 35.238.94.250 80:32404/TCP 15m
```

To check if the result app is correctly running, go to http://{result-service.EXTERNAL-IP} ==> the voting result app must be displayed

You can now add votes in the voting app and check the result in the result app.

## Create the voting app using deployments and services

Deploying PODS does not help to scale the application easily. If a POD fails, it will not come up again.

under deployment folder
kubectl create -f .
kubectl get all

If you're not able to access to the application from the NodeExternalIP/AppExternalPort, please check the firewall conforguration.
You can add a new firewall rule in GCP as follow :
Go to GCP console NETWORKING / VPC network / firewall rules
In the list of rules, copy the target of the rules related to the voting-app
Click on the button create firewall rule
Name : voting app
Targets : specified target tags
Targets tag : gke-example-voting-app-d3ec8c0c-node // past the target you've copied later
Source filter : ip ranges
Source IP ranges : 0.0.0.0/0
protocols and ports : choose specified protocols and ports, check tcp : and mention the front services external ports
