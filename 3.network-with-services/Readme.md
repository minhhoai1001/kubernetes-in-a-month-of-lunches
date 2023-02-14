## How Kubernetes routes network traffic
You can see that if you deploy two Pods—you can ping one Pod
from the other, but you first need to find its IP address.

```
# create two deployments, which each run one Pod:
kubectl apply -f sleep/sleep1.yaml -f sleep/sleep2.yaml

# wait for the Pod to be ready:
kubectl wait --for=condition=Ready pod -l app=sleep-2

# check the IP address of the second Pod:
kubectl get pod -l app=sleep-2 --output jsonpath=‘{.items[0].status.podIP}’

# use that address to ping the second Pod from the first:
kubectl exec deploy/sleep-1 -- ping -c 2 $(kubectl get pod -l app=sleep-2 --
output jsonpath=‘{.items[0].status.podIP}’)
```

These Pods are managed by deployment controllers. If you
delete the second Pod, its controller will start a replacement that has a new IP
address.

```
# check the current Pod’s IP address:
kubectl get pod -l app=sleep-2 --output jsonpath=‘{.items[0].status.podIP}’

# delete the Pod so the deployment replaces it:
kubectl delete pods -l app=sleep-2

# check the IP address of the replacement Pod:
kubectl get pod -l app=sleep-2 --output jsonpath=‘{.items[0].status.podIP}’
```

You deploy a Service using a YAML file and the usual Kubectl
apply command. Deploy the Service and verify the network traffic is routed to
the Pod.

```
# deploy the Service defined in listing 3.1:
kubectl apply -f sleep/sleep2-service.yaml

# show the basic details of the Service:
kubectl get svc sleep-2

# run a ping command to check connectivity—this will fail:
kubectl exec deploy/sleep-1 -- ping -c 1 sleep-2
```
### NOTE: 
The ping command didn’t work as expected because ping uses a network protocol that isn’t supported in Kubernetes Services.

## Routing traffic between Pods
The default type of Service in Kubernetes is called `ClusterIP`—it creates a cluster-wide
IP address that Pods on any node can access. The IP address only works within the
cluster, so ClusterIP Services are only useful for communicating between Pods.

Run two deployments, one for the web application and one for
the API. There are no Services for this app yet, and it won’t work correctly
because the website can’t find the API.

```
# run the website and API as separate deployments:
kubectl apply -f numbers/api.yaml -f numbers/web.yaml

# wait for the Pod to be ready:
kubectl wait --for=condition=Ready pod -l app=numbers-web

# forward a port to the web app:
kubectl port-forward deploy/numbers-web 8080:80

# browse to the site at http://localhost:8080 and click the Go button
#—you’ll see an error message
# exit the port forward:
ctrl-c
```

Create the Service for the API so the domain lookup works and
traffic gets sent from the web Pod to the API Pod.
```
# deploy the Service from listing 3.2:
kubectl apply -f numbers/api-service.yaml

# check the Service details:
kubectl get svc numbers-api

# forward a port to the web app:
kubectl port-forward deploy/numbers-web 8080:80

# browse to the site at http://localhost:8080 and click the Go button
# exit the port forward:
ctrl-c
```
The API Pod is managed by a Deployment controller, so you can
delete the Pod and a replacement will be created. The replacement is also a
match for the label selector in the API Service, so traffic gets routed to the
new Pod and the app keeps working.

```
# check the name and IP address of the API Pod:
kubectl get pod -l app=numbers-api -o custom-
columns=NAME:metadata.name,POD_IP:status.podIP

# delete that Pod:
kubectl delete pod -l app=numbers-api

# check the replacement Pod:
kubectl get pod -l app=numbers-api -o custom-
columns=NAME:metadata.name,POD_IP:status.podIP

# forward a port to the web app:
kubectl port-forward deploy/numbers-web 8080:80

# browse to the site at http://localhost:8080 and click the Go button
# exit the port forward:
ctrl-c
```

## Routing external traffic into Pods
A `LoadBalancer` Service integrates with an external load balancer
which sends traffic into the cluster. The Service sends the traffic
to a Pod—using the same label selector mechanism to identify
a target Pod.

Deploy the Service, and then use Kubectl to find the address of
the Service.
```
# deploy the LoadBalancer Service for the website—if your firewall checks 
# that you want to allow traffic, then its OK to say Yes:
kubectl apply -f numbers/web-service.yaml

# check the details of the Service:
kubectl get svc numbers-web

# use formatting to get the app URL from the EXTERNAL-IP field
kubectl get svc numbers-web -o jsonpath=‘http://{.status.loadBalancer.ingress[0].*}:8080’
```

### NOTE:
The LoadBalancer Service is created with a real IP address. This is a local cluster so
it’s not a public IP address, but if I ran this same exercise in an AKS or EKS cluster
then the Service would have a public address assigned by the cloud provider.

`NodePort` Services have each node listening on the
Service port. There’s no external load balancer so
traffic is routed directly to the cluster nodes.

## Routing traffic outside Kubernetes
`ExternalName` Services let you use local names in your
application Pods, and the DNS server in Kubernetes resolves the local name to a fully-
qualified external name when the Pod makes a lookup request.

You can’t switch a Service from one type to another, so you’ll
need to delete the original ClusterIP Service for the API before you can
deploy the ExternalName Service.

```
# delete the current API Service:
kubectl delete svc numbers-api

#deploy a new ExternalName Service:
kubectl apply -f numbers-services/api-service-externalName.yaml

# check the Service configuration:
kubectl get svc numbers-api

# refresh the website in your browser and test with the Go button
```

## Lab
This lab is going to give you some practice creating Services, but it’s also going to get
you thinking about labels and selectors, which are powerful features of Kubernetes.
The goal is to deploy Services for an updated version of the random number app
which has had a UI makeover. Here are your hints:
- The lab folder for this chapter has a `deployments.yaml` file. Use that to deploy
the app with Kubectl.
- Check the Pods. There are two versions of the web application running.
- Write a Service that will make the API available to other Pods at the domain
name numbers-api.
- Write a Service which will make version two of the website available externally,
on port 8088.
- You’ll need to look closely at the Pod labels to get the correct result.