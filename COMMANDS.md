# Manifest Walkthrough

## Results

### Testing From Inside Cluster

#### Pod

```
root@yuriy-k8s-cluster-vmouaaa7oope-master-0:~# kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE    IP               NODE                                    NOMINATED NODE   READINESS GATES
hello-world-db9bd847-m6ncw   1/1     Running   0          118m   10.100.121.195   yuriy-k8s-cluster-vmouaaa7oope-node-0   <none>           <none>

root@yuriy-k8s-cluster-vmouaaa7oope-master-0:~# curl 10.100.121.195:8080; echo
Hello Ryan! Hello, world!
```

#### ClusterIP

```
root@yuriy-k8s-cluster-vmouaaa7oope-master-0:~# kubectl get services hello-world-clip
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hello-world-clip   ClusterIP   10.254.46.160   <none>        80/TCP    67m
root@yuriy-k8s-cluster-vmouaaa7oope-master-0:~# curl 10.254.46.160; echo
Hello Ryan! Hello, world!
```

### Testing from Outside the Cluster

#### NodeIPs

```
ubuntu@yuriyjs:~$ kubectl get nodes -o wide
NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                        KERNEL-VERSION          CONTAINER-RUNTIME
yuriy-k8s-cluster-vmouaaa7oope-master-0   Ready    master   3h41m   v1.23.3   10.0.0.110    <none>        Fedora CoreOS 39.20231204.3.3   6.6.3-200.fc39.x86_64   docker://24.0.5
yuriy-k8s-cluster-vmouaaa7oope-node-0     Ready    <none>   3h38m   v1.23.3   10.0.0.184    <none>        Fedora CoreOS 39.20231204.3.3   6.6.3-200.fc39.x86_64   docker://24.0.5
ubuntu@yuriyjs:~$ curl 10.0.0.110:30080; echo; curl 10.0.0.184:30080; echo
Hello Ryan! Hello, world!
Hello Ryan! Hello, world!
```

#### LoadBalancer (Bonus: domain name)

```
ubuntu@yuriyjs:~$ kubectl get services hello-world-lb
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)        AGE
hello-world-lb   LoadBalancer   10.254.140.113   173.231.254.251   80:32741/TCP   80m
ubuntu@yuriyjs:~$ dig +short aweber.yuriys.net
173.231.254.251
ubuntu@yuriyjs:~$ curl 173.231.254.251; echo
Hello Ryan! Hello, world!
ubuntu@yuriyjs:~$ curl http://aweber.yuriys.net; echo
Hello Ryan! Hello, world!
```

## Deployment Walkthrough


```
yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   1/1     1            1           114m
yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl describe deployments hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Fri, 12 Jan 2024 00:00:46 -0500
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-world
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-world
  Containers:
   nginx:
    Image:        nginx:alpine
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /etc/nginx/conf.d from nginx-config (rw)
  Volumes:
   nginx-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      nginx-config
    Optional:  false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-db9bd847 (1/1 replicas created)
Events:          <none>
```

## Service Walkthrough

```
yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl get services
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)        AGE
hello-world-clip   ClusterIP      10.254.46.160    <none>            80/TCP         76m
hello-world-lb     LoadBalancer   10.254.140.113   173.231.254.251   80:32741/TCP   76m
hello-world-np     NodePort       10.254.121.136   <none>            80:30080/TCP   24m
kubernetes         ClusterIP      10.254.0.1       <none>            443/TCP        3h38m

yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl describe services hello-world-clip
Name:              hello-world-clip
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=hello-world
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.254.46.160
IPs:               10.254.46.160
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.100.121.195:8080
Session Affinity:  None
Events:            <none>

yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl describe services hello-world-lb
Name:                     hello-world-lb
Namespace:                default
Labels:                   <none>
Annotations:              loadbalancer.openstack.org/load-balancer-id: 6ef598fa-b793-4a1a-acee-172d7ae9c4fb
Selector:                 app=hello-world
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.254.140.113
IPs:                      10.254.140.113
LoadBalancer Ingress:     173.231.254.251
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32741/TCP
Endpoints:                10.100.121.195:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                  Age                  From                Message
  ----     ------                  ----                 ----                -------
  Warning  SyncLoadBalancerFailed  60m                  service-controller  Error syncing load balancer: failed to ensure load balancer: load balancer 6ef598fa-b793-4a1a-acee-172d7ae9c4fb is not ACTIVE, current provisioning status: PENDING_CREATE
  Normal   EnsuringLoadBalancer    9m20s (x3 over 61m)  service-controller  Ensuring load balancer
  Normal   EnsuredLoadBalancer     8m59s (x2 over 60m)  service-controller  Ensured load balancer

yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl describe services hello-world-np
Name:                     hello-world-np
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=hello-world
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.254.121.136
IPs:                      10.254.121.136
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                10.100.121.195:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

## ConfigMap Walkthrough

```
yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      3h24m
nginx-config       1      117m
yuriys@jumpstation:~/Documents/aweber/k8s-nginx-manifest$ kubectl describe configmaps nginx-config
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx.conf:
----
server {
  listen 8080;
  location / {
    types {}
    default_type text/plain;
    add_header Content-Type text/plain;

    return 200 "Hello Ryan! Hello, world!\n";
  }
}


BinaryData
====

Events:  <none>
```