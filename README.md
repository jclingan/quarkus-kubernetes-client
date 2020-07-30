# quarkus-kubernetes-client

This project demonstrates the Quarkus kubernetes client API.
There are two endpoints.
1. */pods*. Returns the list of pods running in the same namespace.
2. */pods/stream*. Streams pod events as they are added, modified, and deleted.

## Prerequisites

* Quarkus 1.6.1.Final or later (because that's the version I've tested)
* minikube 1.11.0 or later (because that's the version I've tested)
* JDK 11 (because that's the version I've tested)
* Maven 3.6.2+

## Instructions

Deploy the application:

```
mvn \
   clean \
   install \
   -DskipTests \
   -Dquarkus.kubernetes.deploy=true
```

Access the endpoint to verify successful deployment,
which should return the list of pods in the namespace:

```
MINIKUBE_IP=`minikube ip`
NODEPORT=`kubectl get svc quarkus-kubernetes-client -o jsonpath='{.spec.ports[0].nodePort}'`

curl http://$MINIKUBE_IP:$NODEPORT/kube/pods
```

Output (Something similar to):
```
[quarkus-kubernetes-client-58f46c4879-g4rcx]
```

Connect to the streamed output, which will block:
```
echo http://$MINIKUBE_IP:$NODEPORT/kube/stream
```

Output (Something similar to):
```
[quarkus-kubernetes-client-58f46c4879-g4rcx]
```
Open the output of the `echo http://$MINIKUBE_IP:$NODEPORT/health-ui` command in the browser. This should show similar output to:
```
data: ADDED: quarkus-kubernetes-client-58f46c4879-g4rcx
```

Open a new terminal window and type thee following:

```
kubectl scale deployment quarkus-kubernetes-client --replicas 3
```

Output in the first terminal should be similar to:
```
data: ADDED: quarkus-kubernetes-client-58f46c4879-hjvvs

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-hjvvs

data: ADDED: quarkus-kubernetes-client-58f46c4879-dgq82

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-dgq82

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-hjvvs

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-dgq82

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-dgq82

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-hjvvs
```

Reduce the number of replicas back to 1:

```
kubectl scale deployment quarkus-kubernetes-client --replicas 1
```

Output in the first terminal should be similar to:
```
data: MODIFIED: quarkus-kubernetes-client-58f46c4879-hjvvs

data: DELETED: quarkus-kubernetes-client-58f46c4879-hjvvs

data: MODIFIED: quarkus-kubernetes-client-58f46c4879-dgq82

data: DELETED: quarkus-kubernetes-client-58f46c4879-dgq82
```

When the connection dies the UI should show the service being _DOWN_.
Once the service is back up, the health-UI showld show a status of  _UP_.
