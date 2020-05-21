# How to deploy Kong
 
 ## Prerequisites
 > Kong supports the following database versions:
 > - **PostgreSQL**: 9.5 and above.
 > - **Cassandra**: 2.2 and above.

 ## Installing Kong for Kubernetes
Note: 
> Do not recommend using Kong with Cassandra on Kubernetes deployments. Should be use managed database service from cloud-provider.

### Prepare ConfigMap for custom plugins
1. Install Luarocks: [This link](..\luarocks\README.md)
2. Install plugin using luarocks. Ex: `luarocks install kong-oidc`
3. Create ConfigMap 

Go to "C:\lua\share\plugins". Then

        kubectl create configmap kong-plugin-oidc --from-file=.\plugins\oidc -n kong  



 ### Yaml manifest
  To deploy Kong via kubectl, use:
  `kubectl apply -f https://bit.ly/kong-ingress-dbless`

 ### Helm Chart
 About Kong chart: [README](https://github.com/Kong/charts/tree/master/charts/kong)

    $ helm repo add kong https://charts.konghq.com
    $ helm repo update

    # Helm 2
    $ helm install kong/kong -f .\kong\kong-value.yaml --name my-kong --namespace kong

    # Helm 3
    $ helm install kong/kong --generate-name --set ingressController.installCRDs=false
### Kustomize
    kustomize build github.com/kong/kubernetes-ingress-controller/deploy/manifests/base

### Upgrade Kong use Helm 2
    $ helm upgrade my-kong kong/kong -f .\kong\kong-value.yaml