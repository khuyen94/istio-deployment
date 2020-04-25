Commands to quickly set up the cluster with the application.

0.  Create 'istio-system' namespace

```bash
kubectl create ns istio-system
```

1.	Install all the Istio Custom Resource Definition(CRDs).

```bash
helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
```

2. Wait for all Istio CRDs to be created.

```bash
kubectl -n istio-system wait --for=condition=complete job –all
```

3. Switch to istio installation dir and install istio into the cluster. This template only support Helm v2.x

```bash

helm template install/kubernetes/helm/istio \
  --set gateways.istio-ingressgateway.serviceAnnotations.'service\.beta\.kubernetes\.io/azure-load-balancer-internal'="true" 
  --set gateways.istio-ingressgateway.serviceAnnotations.'service\.beta\.kubernetes\.io/azure-load-balancer-internal-subnet'="int-core-dev-subnet-services"
  --set global.mtls.enabled=false \
  --set tracing.enabled=true \
  --set kiali.enabled=true \
  --set grafana.enabled=true \
  --namespace istio-system > istio.yaml

kubectl apply -f istio.yaml

# Wait until pods are in Running or Completed state
kubectl get pods -n istio-system

```

4. Label the **default** namespace for auto sidecar injection

```bash

kubectl label namespace default istio-injection=enabled

```

5. Set up the application (Optinal). You can clone this repo: https://github.com/rinormaloku/istio-mastery/

```bash

kubectl apply -f ./resource-manifests/kube

```

6. Set up the Gateway for ingress HTTP requests

```bash

kubectl apply -f resource-manifests/istio/http-gateway.yaml 

```

7. Install the virtual services so that requests are routed

```bash

kubectl apply -f resource-manifests/istio/sa-virtualservice-external.yaml


```

8. Open the application on the IP returned by:

```bash

kubectl get svc -n istio-system \
  -l app=istio-ingressgateway \
  -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}'

```

X. What we get out of the box:

1. Forward port.

```bash
 # Kiali
kubectl port-forward \
    $(kubectl get pod -n istio-system -l app=kiali \
    -o jsonpath='{.items[0].metadata.name}') \
    -n istio-system 20001


# Grafana
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=grafana \
    -o jsonpath='{.items[0].metadata.name}') 3000


# Jaeger 
kubectl port-forward -n istio-system \
    $(kubectl get pod -n istio-system -l app=jaeger \
    -o jsonpath='{.items[0].metadata.name}') 16686
```

2. Expose via ingress gateway.

```bash
# Grafana
$ cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: grafana-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 15031
      name: http-grafana
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: grafana-vs
  namespace: istio-system
spec:
  hosts:
  - "*"
  gateways:
  - grafana-gateway
  http:
  - match:
    - port: 15031
    route:
    - destination:
        host: grafana
        port:
          number: 3000
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: grafana
  namespace: istio-system
spec:
  host: grafana
  trafficPolicy:
    tls:
      mode: DISABLE
---
EOF
gateway.networking.istio.io "grafana-gateway" configured
virtualservice.networking.istio.io "grafana-vs" configured
destinationrule.networking.istio.io "grafana" configured

# Kiali
$ cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: kiali-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 15029
      name: http-kiali
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: kiali-vs
  namespace: istio-system
spec:
  hosts:
  - "*"
  gateways:
  - kiali-gateway
  http:
  - match:
    - port: 15029
    route:
    - destination:
        host: kiali
        port:
          number: 20001
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: kiali
  namespace: istio-system
spec:
  host: kiali
  trafficPolicy:
    tls:
      mode: DISABLE
---
EOF
gateway.networking.istio.io "kiali-gateway" configured
virtualservice.networking.istio.io "kiali-vs" configured
destinationrule.networking.istio.io "kiali" configured

# Prometheus
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: prometheus-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 15030
      name: http-prom
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: prometheus-vs
  namespace: istio-system
spec:
  hosts:
  - "*"
  gateways:
  - prometheus-gateway
  http:
  - match:
    - port: 15030
    route:
    - destination:
        host: prometheus
        port:
          number: 9090
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: prometheus
  namespace: istio-system
spec:
  host: prometheus
  trafficPolicy:
    tls:
      mode: DISABLE
---
EOF

# Jaeger
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: tracing-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 15032
      name: http-tracing
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tracing-vs
  namespace: istio-system
spec:
  hosts:
  - "*"
  gateways:
  - tracing-gateway
  http:
  - match:
    - port: 15032
    route:
    - destination:
        host: tracing
        port:
          number: 80
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: tracing
  namespace: istio-system
spec:
  host: tracing
  trafficPolicy:
    tls:
      mode: DISABLE
---
EOF

#For secure access, follow this article: https://istio.io/docs/tasks/observability/gateways/ 

```