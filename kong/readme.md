# How to deploy Kong
 
 ## Prerequisites
 > Kong supports the following database versions:
 > - **PostgreSQL**: 9.5 and above.
 > - **Cassandra**: 2.2 and above.

 ## Installing Kong for Kubernetes
Note: 
> Do not recommend using Kong with Cassandra on Kubernetes deployments. Should be use managed database service from cloud-provider.

 ### Yaml manifest
  To deploy Kong via kubectl, use:
  `kubectl apply -f https://bit.ly/kong-ingress-dbless`

 ### Helm Chart
    helm repo add kong https://charts.konghq.com
    $ helm repo update

    # Helm 2
    $ helm install kong/kong

    # Helm 3
    $ helm install kong/kong --generate-name --set ingressController.installCRDs=false
### Kustomize
    kustomize build github.com/kong/kubernetes-ingress-controller/deploy/manifests/base
