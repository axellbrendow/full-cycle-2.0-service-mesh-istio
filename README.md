# full-cycle-2.0-service-mesh-istio

Files I produced during the "Service Mesh with Istio" classes of my [Microservices Full Cycle 3.0 course](https://drive.google.com/file/d/1bJnFxQPKgSsI30sCvW-KzYK4V5JWzgSs/view?usp=share_link).

# Links

- [k3d / Getting Started](https://k3d.io/)
- [Istio / Getting Started](https://istio.io/latest/docs/setup/getting-started/)

# Install Istio on k8s cluster

```sh
istioctl install -y
```

# Install Grafana, Jaeger, Kiali and Prometheus addons on Istio

```sh
kubectl apply -f https://github.com/istio/istio/raw/release-1.10/samples/addons/grafana.yaml
kubectl apply -f https://github.com/istio/istio/raw/release-1.10/samples/addons/jaeger.yaml
kubectl apply -f https://github.com/istio/istio/raw/release-1.10/samples/addons/kiali.yaml
kubectl apply -f https://github.com/istio/istio/raw/release-1.10/samples/addons/prometheus.yaml
```

## Open kiali dashboard

```sh
istioctl dashboard kiali
```

## Fire requests to the k8s service to see the traffic in Kiali dashboard

```sh
while true
do      
    curl http://localhost:8000
    sleep 0.5
done
```

## Using Fortio, a load testing library, to fire requests

```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.10/samples/httpbin/sample-client/fortio-deploy.yaml
export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- fortio load -c 2 -qps 0 -t 200s -loglevel Warning http://nginx-service:8000
```

## Using stick sessions and consistent hash so that users are kept on the same version

```sh
kubectl apply -f consistent-hash.yaml
POD=$(kubectl get pods | grep nginx-a | head -n1 | cut -d' ' -f1)
kubectl exec -it $POD -- bash

# Then, inside the pod
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
curl --header "x-user: user1" http://nginx-service:8000
# Note that all requests will get the same response

curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
curl --header "x-user: user2" http://nginx-service:8000
# Note that all requests will get the same response

curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
curl --header "x-user: user3" http://nginx-service:8000
# Note that all requests will get the same response
```

## Fault Injection through Virtual Service

```sh
kubectl apply -f fault-injection.yaml
```

## Creating an istio ingress gateway (a gateway that contains an istio proxy)

If you're creating a cluster in the cloud, go to "Finally, you can:".

When I created my local cluster, I run:

`k3d cluster create -p "8000:30000@loadbalancer" --agents 2`

So, when I access localhost:8000 I'm redirected to the port 30000 in my cluster,
this way, I can access the LoadBalancer of `deployment.yaml`.

But now, we wanna access localhost:8000 and get redirected to the istio-ingressgateway. First, open `deployment.yaml` and edit the LoadBalancer nodePort to 30001 to let port 30000 free.

Now, run `kubectl get svc -n istio-system` and discover the NodePort of **istio-ingressgateway**:

```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
istio-ingressgateway   LoadBalancer   10.43.31.68     <pending>     15021:30847/TCP,80:31784(<---- this one)/TCP
```

Then, edit the **istio-ingressgateway** service NodePort:

```sh
kubectl edit svc istio-ingressgateway -n istio-system
# Search for:
# - name: http2
#   nodePort: 31784 # Change 31784 to 30000 and save
#   port: 80
```

Finally, you can:

```sh
kubectl apply -f gateway.yaml
```

## Creating istio gateway based on subdomains

Just remember to add "a.fullcycle.com" and "b.fullcycle.com" to /etc/hosts and then:

```sh
kubectl apply -f gateway-subdomains.yaml
```
