# How to install Cert Manager
https://cert-manager.io/docs/installation/kubernetes/

## 1. Create CA cluster issuer
    kubectl apply -f cluster-issuer.yaml

    https://github.com/fbeltrao/aks-letsencrypt/blob/master/setup-wildcard-certificates-with-azure-dns.md