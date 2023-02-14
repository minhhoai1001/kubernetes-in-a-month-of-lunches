## Defining Deployments in application manifests

```
# deploy the application from the manifest file:
kubectl apply -f pod.yaml

# list running Pods:
kubectl get pods
```
Kubectl’s port forward command sends traffic into a Pod
```
# run a port forward from your local machine to the Deployment:
kubectl port-forward hello-kiamol-3 8080:80

# browse to http://localhost:8080
# when you’re done, exit with ctrl-c
```

Apply the Deployment manifest which will create a new Deploy-
ment, which in turn will create a new Pod.

```
# run the app using the Deployment manifest:
kubectl apply -f deployment.yaml

# find Pods managed by the new Deployment:
kubectl get pods -l app=hello-kiamol-4
```

You can run commands inside containers with Kubectl
```
# check the internal IP address of the first Pod we ran:
kubectl get pod hello-kiamol-3 -o custom-
columns=NAME:metadata.name,POD_IP:status.podIP

# run an interactive shell command in the Pod:
kubectl exec -it hello-kiamol-3 -- sh

# inside the Pod check the IP address:
hostname -i

# and test the web app:
wget -O—http://localhost | head -n 4

# leave the shell:
exit
```

Use the Kubectl delete command to remove all Pods, and check
that they’re really gone.
```
# list all running Pods:
kubectl get pods

# delete all Pods:
kubectl delete pods --all

# check again:
kubectl get pods
```

Check the Deployments you have running, then delete them
and check the remaining Pods have been deleted.
```
# view Deployments:
kubectl get deploy

# delete all Deployments:
kubectl delete deploy --all

# view Pods:
kubectl get pods

# check all resources:
kubectl get all
```

## Lab
This is your first lab—it’s a guided challenge for you to complete yourself. The goal is
to write a Kubernetes YAML spec for a Deployment which will run an application in a
Pod, then test the app and make sure it runs as expected. Here are a few hints to get
you started:
- In the ch02/lab folder, there’s a file called pod.yaml which you can try out—it
runs the app but it defines a Pod rather than a Deployment.
- The application container runs a website that listens on port 80.
- When you forward traffic to the port, the web app responds with the hostname
of the machine it’s running on.
- That hostname is actually the Pod name, which you can verify using Kubectl.