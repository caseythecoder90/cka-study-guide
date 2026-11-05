# Services

Creating, inspecting and debugging Services. Chapter: [02-14 Services](../02-core-concepts/14-services.md). How the rules work on a node: [`kube-proxy.md`](kube-proxy.md).

## Create

```bash
# Selector copied from the Deployment; no --node-port flag
kubectl expose deployment myapp --port=80 --target-port=8080 --type=NodePort --name=myapp-service
kubectl expose deployment myapp --port=80 --type=NodePort --name=myapp-service --dry-run=client -o yaml > svc.yaml

# Selector set to app=<service-name>; fix it if your Pods use other labels
kubectl create service nodeport myapp-service --tcp=80:8080 --node-port=30008 --dry-run=client -o yaml > svc.yaml
kubectl create service clusterip backend --tcp=80:8080
kubectl create service externalname my-service --external-name=my.database.example.com
kubectl apply -f svc.yaml
```

## Inspect

```bash
kubectl get svc
kubectl get svc myapp-service -o wide            # + SELECTOR
kubectl describe svc myapp-service
kubectl get endpointslices -l kubernetes.io/service-name=myapp-service
kubectl get svc myapp-service -o jsonpath='{.spec.ports[0].nodePort}{"\n"}'
kubectl get nodes -o wide                        # node IPs to curl
```

## Debug, in order

```bash
kubectl get endpointslices -l kubernetes.io/service-name=myapp-service   # empty = selector or readiness
kubectl get svc myapp-service -o jsonpath='{.spec.selector}{"\n"}'
kubectl get pods --show-labels                                           # do the labels match?
kubectl get pods -o wide                                                 # Ready? Same namespace?
kubectl describe svc myapp-service | grep -i targetport                  # the port the app listens on?
curl http://<node-ip>:30008
kubectl run tmp --image=busybox --rm -it --restart=Never -- wget -qO- -T 3 myapp-service
```

## ExternalName

```bash
kubectl get svc my-service                       # EXTERNAL-IP shows the DNS name; no CLUSTER-IP
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup my-service
```

## ClusterIP

Chapter: [02-15 ClusterIP Services](../02-core-concepts/15-services-clusterip.md).

```bash
kubectl expose deployment back-end --port=80 --target-port=8080   # ClusterIP is the default
kubectl create service clusterip back-end --tcp=80:8080           # selector is app=back-end
kubectl create service clusterip db --clusterip=None              # headless
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup back-end        # same namespace
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup back-end.prod   # another namespace
kubectl run tmp --image=busybox --rm -it --restart=Never -- wget -qO- -T 3 http://back-end:80
kubectl -n kube-system get pods -l k8s-app=kube-dns               # CoreDNS
```

## LoadBalancer

Chapter: [02-16 LoadBalancer Services](../02-core-concepts/16-services-loadbalancer.md).

```bash
kubectl expose deployment myapp --port=80 --type=LoadBalancer --name=myapp-service
kubectl create service loadbalancer myapp-service --tcp=80:80    # selector is app=myapp-service
kubectl get svc myapp-service -w                                 # EXTERNAL-IP: <pending>, then an address
kubectl get svc myapp-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}{"\n"}'
kubectl get svc myapp-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}{"\n"}'   # some providers
kubectl get svc myapp-service -o jsonpath='{.spec.ports[0].nodePort}{"\n"}'
curl http://<node-ip>:<node-port>                                # no cloud integration needed
```
