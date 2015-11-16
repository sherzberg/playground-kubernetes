# kubernetes playground

This project assumes you have an [AWS Kubernetes Environment Setup](http://kubernetes.io/v1.1/docs/getting-started-guides/aws.html).

The `yaml` files in the project will be used to play with Kubernetes on AWS.

## Replication Controller

Replication controllers keep a certain number of Pods running. If one does, Kubernetes will start another.

Lets create a RC that just runs a simple webapp:

```bash
$ kubectl create -f frontend-rc.yaml
```

And lets check out the status:

```bash
$ kubectl get rc
CONTROLLER   CONTAINER(S)   IMAGE(S)          SELECTOR       REPLICAS
frontend     frontend       deis/helloworld   app=frontend   2
```

## ELB Service

Services tie pods to externally (and internally) identifiable names. By default Pods are not exposed
to external address. Services provide ways to expose your Pods services.

Kubernetes has a few different ways to create Services, but when on AWS, we can have
a service that sets up an Elastic Load Balancer tied to our replication controller pods. This
is super slick. Other cloud platforms that have concepts similar to ELBs may have similar outcomes.

```bash
$ kubectl create -f frontend-svc.yaml
```

And to see the progress:

```bash
$ kubectl get services
NAME          LABELS                                    SELECTOR       IP(S)         PORT(S)
frontendsvc   app=frontend                              app=frontend   10.0.25.103   80/TCP
kubernetes    component=apiserver,provider=kubernetes   <none>         10.0.0.1      443/TCP
```

Now lets check out the ELB:

```bash
$ kubectl describe svc frontend
Name:                   frontendsvc
Namespace:              default
Labels:                 app=frontend
Selector:               app=frontend
Type:                   LoadBalancer
IP:                     10.0.99.999
LoadBalancer Ingress:   afffffffffffffffff-1664285561.us-east-1.elb.amazonaws.com
Port:                   <unnamed>       80/TCP
NodePort:               <unnamed>       32120/TCP
Endpoints:              10.244.1.7:80,10.244.1.8:80
Session Affinity:       None
No events.
```

It may take a bit to get a `LoadBalancer Ingress CNAME` but be patient. When you get one, you may curl it!

```bash
$ curl afffffffffffffffff-1664285561.us-east-1.elb.amazonaws.com
Welcome to Deis!
See the documentation at http://docs.deis.io/ for more information.
```

__note__ the docker webserver we use is just a dummy webapp from the Deis project.

## DNS Service Discovery

Another way to access services inside the Kubernetes cluster is via DNS. Kubernetes provides
`kubedns`, so as long as that is running, our pods can access other services via dns:

Lets create a pod that we can remote execute commands through:

```bash
kubectl create -f curlpod.yaml
```

Now lets run an `nslookup` on our `frontendsvc`;

```bash
$ kubectl exec curlpod -- nslookup frontendsvc
Server:    10.0.0.10
Address 1: 10.0.0.10 ip-10-0-0-10.us-west-2.compute.internal

Name:      frontendsvc
Address 1: 10.0.25.103 ip-10-0-25-103.us-west-2.compute.internal
```

Any just for good measure:

```bash
$ kubectl exec curlpod -- curl -s frontendsvc
Welcome to Deis!
See the documentation at http://docs.deis.io/ for more information.
```
