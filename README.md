# Kubernetes type objects

- Node
- Deployment
- pod
- service

## Creating Kubernetes Cluster with kind

```bash
:~$ kind create cluster --config kind-multi-node.yaml
:~$ kubectl get nodes
:~$ docker ps
:~$ kind describe node kind-control-plane
```

Unlike minikube, where one use minikube ssh to log into the node if you want to explore the processes running inside it,
with kind you use "docker exec"

## Creating a kubernetes cluster with minikube

```bash
:~$ minikube start
:~$ minikube status
```

## Imperative way of Deploying your application

```bash
:~$ kubectl create deployment kiada --image=okoro/kiada:0.1 --port=8080
```

The command above specifies three things:

- You want to create a deployment object
- You want the object to be called kiada
- You want the deployment to use the container image okoro/kiada:0.1
- The pod will be exposed to port metadata inside the pod spec

## Listing Deployments currently existing the the cluster

```bash
:~$ kubectl get deployments
:~$ kubectl get pods
```

In kubernetes, we deploy group of col-located containers called pods.
A pod is a group of one or more closely related containers that run together on the same worker node and need to share certain Linux namespaces, so that they can intereact more closely than with other pods.

A node can have one or more pods, while a pod may have one or more containers. You can think of a pod as a separate logical computer that contains one application. The application process and additional supporting processes, each running in a separate conatainer.
Pods are distributed across all the worker nodes of the cluster. 

## How creating a deployment results in a running application container

1. You run kubectl create deployment
2. kubectl creates the deployment object in the kubernetes API, by sending an HTTP request to the Kubernetes API
3. kubernetes then creates a pod object based on the deployment.
4. The pod is assigned to a worker node
5. The kubelet in the worker node instructs the docker runtime to pull the docker image and run the container
6. The application will run run in the container

**NOTE:** Unlike the OS process, once a pod is assigned to a node, it runs only on that node. Even if it fails, this instance of the pod is never moved to other nodes, as is the case with CPU processes.

## Exposing your application to the world

- Each pods gets its own IP address, but this address is internal to the cluster and not accessible from the outside.
- pods are only externally accessible when exposed by creating a service object.
- There are different types of services (ClusterIP Service, NodePort Service, LoadBalancer Service(Used for external communication))

## Creating LoadBalancer Service the imperative way

Not all kubernetes clusters have have the mechanism to provide a loadbalancer. The cluster provided by minikube and kind are examples of such. kubectl always shows the external IP as "pending", and you must use a different method to access the service.
Several methods of accessing services exist. You can even bypass the service and access individual pods directly, but this should only be used for troubleshooting.  

```bash
:~$ kubectl expose deployment kiada --type=LoadBalancer --port 8080
:~$ kubectl get services 
```

The above command does the followings:

1. Exposes all pods that belong to the kiada Deployment as a new service
2. Makes the pods accessible from outside the cluster via a loadbalancer
3. The application listens on port 8080 
4. Since we did not specify a name for the service object, it inherits the name of the deployment.

**NOTE:** 
If using minikube or kind, they do not create a loadbalancer IP. You will need to do the followings:

### Minikube

minikube requires that you create a tunnel, and only then would you be able to get an IP for the loadbalance service: 

```bash
:~$ minikube tunnel
```

<<<<<<< HEAD
But one can still access the application via port forwarding.
=======
## kubectl config priority order

1. --kubeconfig flag (highest priority)
```bash
:~$ kubectl get pods --kubeconfig=/path/to/file.yaml
```

2. KUBECONFIG environment variable
```bash
:~$ export KUBECONFIG=/path/to/file.yaml
:~$ kubectl get pods
```

3. Default location ~/.kube/config (lowest priority, fallback)
>>>>>>> 484253b (Added helm for kubernete package managemant)

```bash
:~$ minikube service kiada --url
```

This is the IP of the minikube VM. The service is accessible via the same port number on all your worker nodes, regardless of whethe you're using minikube or any other kubernetes cluster.

## Horizontal Scalling

With kubernetes, scaling out an application, i.e horizontal scalling is trivial to do. To run additional instances, you only need to scale the Deployment object with the following command. 

```bash
:~$ kubectl scale deployment kiada --replica=3
```

This is one of the most fundamental principles in kubernetes. Instead of telling kubernetes what to do, you simply set a new desired state of the system and let kubernetes achieve it. This commands actually modifies the Deployment object.
