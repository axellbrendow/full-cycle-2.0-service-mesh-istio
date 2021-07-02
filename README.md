# full-cycle-2.0-kubernetes

Files I produced during the "Service Mesh with Istio" classes of the Full Cycle 2.0 [course](https://drive.google.com/file/d/1MdN-qK_8Pfg6YI3TSfSa5_2-FHmqGxEP/view?usp=sharing)

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

