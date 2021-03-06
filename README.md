# This is Kubernetes learning repo

_This exercise is using images from Docker public repository_

## How deploy application out of public repository
_(this method creates kubernetes service to enable public access to our POD)_
1. create deployment
```
$kubectl run hello --image=gcr.io/google-samples/hello-go-gke:1.0
```
2. expose it through k8s service
```
$kubectl expose deployments hello --port 80 --type LoadBalancer
```
3. describe service you just created to get host IP
```
$kubectl describe service hello
```
use {kubernetes_cluseter_ip}:{node_port} to access your service

## How create simple hello POD to be able to call it with http://127.0.0.1:10800
_(still don’t know why kubernetes wants 10K port range)_
_(this method configures port forwarding to enable public access to our POD)_
Pod can be created with configuration file (YAML)
1. create configuration file (example in hello.yaml)
2. create POD out of that configuration file
```
$kubectl create -f hello.yaml
```
3. describe just created POD, to get POD IP address and see event log
```
$kubectl describe pod hello
```
4. you cannot access POD directly, you have to do port-forward
```
$kubectl port-forward hello 10080:80
```
in another terminal you can execute
```
curl 127.0.0.1:10080
```
5. see logs from your application
```
$kubectl logs hello
$kubectl logs -f hello
```
6. if you need to sneak-peek in your POD open interactive shell (for example to ping from POD outside)
```
$kubectl exec hello --stdin --tty -c shell_name /bin/sh
```
Don't forget to exit interactive shell

## How to access your POD via TLS
We use here nginx as reverse-proxy configured to forward traffic from 443 to our hello port 80. This example is using another YAML file, because we are going to mount TSL certificates and nginx configurations as volumes in our POD, so both hello and nginx containers can access them. While both containers are packed in same POD they share same namespace and can communicate with each other. TSL and nginx configurations are stored in kubernetes secrets and configmaps.
1. create kubectl secret
```
$kubectl create secret generic tls-certs --from-file=tls
$kubectl describe secrets tls-certs
```
2. create kubectl configmap
```
$kubectl create configmap nginx-proxy-conf --from-file proxy.conf
$kubectl describe configmap nginx-proxy-conf
```
3. create POD from hellosecured.yml
```
$kubectl create -f hellosecured.yml
```
after couple of seconds you can see the status of your deployment
```
$kubectl get pod --output=yaml
```
4. do port-forwarding in one terminal
```
$kubectl port-forward hello 10443:443
```
5. in another terminal you can call
```
$curl -k https://127.0.0.1:10443
```
and you get
```
{“message":"Hello"}
```

## How access your POD directly through Cluster Load balancer
Normally you don't want to do port-forwarding all the time to access POD. This is also not viable, since you have to take care of your 
Node IP address and reconfigure port-forwarding every time you deploy POD new. It is much easier to access POD from outside if you let
k8s cluster load balancer do the job for you. In order to benefit here, you have to expose your POD as a k8s service.
1. create hello_service.yml to create service and execute
```
$kubectl create -f hello_service.yml
```
2. all these calls are going to fail
direct call to cluster IP
```
$curl -k https://10.0.0.63:31000
```
direct call to k8s cluster load balancer
```
curl -k https://192.168.99.100:31000
```
These calls fail because we have no label's set on our hello POD
3. set same labels on POD that you used in spec.selector part of your hello_service.yml
```
$kubectl label pods hello "app=hello"
$kubectl label pods hello "secure=enabled"
$kubectl get pods -l "app=hello"
$kubectl get pods -l "secure=enabled"
```
4. now you can call your hello service
```
$curl -k https://192.168.99.100:31000
```


