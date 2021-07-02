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
