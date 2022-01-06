Follow steps from here: https://www.architect.io/blog/kafka-docker-tutorial

Start everyting:
- Zookeeper
- Kafka
- Publisher
- Consumer

```
 docker-compose up
```

## Build and push to Docker Registry
---
**NOTE**
--tag , -t		Name and optionally a tag in the 'name:tag' format
---

```
docker build .\publisher -t mckeepa/publisher:latest
docker build .\subscriber -t mckeepa/subscriber:latest

docker push mckeepa/publisher:latest
docker push mckeepa/subscriber:latest
```

## Start Kubernetes DashBoard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
```

## Creating a Service Account

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

Create a service account named dashboard-admin-sa in the default namespace
```
kubectl create serviceaccount dashboard-admin-sa
```
Bind the dashboard-admin-service-account service account to the cluster-admin role
```
kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa
```

When we created the dashboard-admin-sa service account Kubernetes also created a secret for it.
List secrets using:
```
kubectl get secrets
```
Use kubectl describe to get the access token:
```
kubectl describe secret dashboard-admin-sa-token-2dqrg
```


Load the Dashboard:
``` url
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/deployment?namespace=kafka-example
```
We are creating Service Account with the name admin-user in namespace kubernetes-dashboard first.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```


kubectl apply -f dashboard-adminuser.yaml
kubectl apply -f dashboard-cluster-role-binding.yaml

kubectl -n kubernetes-dashboard get sa/admin-user
kubectl -n kubernetes-dashboard get secret  

kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

kubectl proxy

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.

# Kubernetes setup
kubectl create -f ./kubernetes/namespace.yml 
kubectl create -f ./kubernetes/kafka-service.yml
kubectl create -f .\kubernetes\zookeeper-service.yml
kubectl create -f .\kubernetes\zookeeper-deployment.yml
kubectl create -f .\kubernetes\kafka-deployment.yml  
kubectl create -f .\kubernetes\publisher-deployment.yml
kubectl create --save-config -f .\kubernetes\subscriber-deployment.yml

or

kubectl create -f namespace.yml --kubeconfig=<kubeconfig_file_for_your_cluster> 


## Clean up 
kubectl delete namespace kafka-example --kubeconfig=<kubeconfig_file_for_your_cluster>

## Get the proxy mode on one of the nodes (kube-proxy listens on port 10249):
``` bash
# Run this in a shell on the node you want to query.
curl http://localhost:10249/proxyMode
```

