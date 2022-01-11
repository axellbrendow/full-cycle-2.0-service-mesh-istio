# full-cycle-2.0-kubernetes

Files I produced during the "Service Mesh with Istio -> Circuit Breaker" classes of my [Microservices Full Cycle 2.0 course](https://drive.google.com/file/d/1MdN-qK_8Pfg6YI3TSfSa5_2-FHmqGxEP/view?usp=sharing).

# Build and push go application image

```sh
cd servicex
docker build -t axell13/go-circuit-breaker:latest .
docker push axell13/go-circuit-breaker
```

# Apply deployment to the k8s cluster

```sh
kubectl apply -f k8s/deployment.yaml
export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://servicex-service

# Note that 50% of the requests will fail and 50% will be successful
```

# Apply circuit breaker to the k8s cluster

```sh
kubectl apply -f k8s/circuit-breaker.yaml
kubectl exec "$FORTIO_POD" -c fortio -- fortio load -c 2 -qps 0 -n 200 -loglevel Warning http://servicex-service

# Note that 10 requests will fail and the remaining will be successful. That's because the circuit breaker stops sending the traffic to the pod that's failing and send to the other pods.
```
