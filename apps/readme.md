# A/B Testing — Destination Rules
When have two versions of an application, and we aren’t 100% sure which will increase user interaction, so we try both versions at the same time and collect metrics.

## 1. Deploy two version

## 2. Deploy DestinationRules 
Use DestinationRules to specify policy that apply to the traffic intended for the instances of this Service.

    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: comvita-mysupply
    spec:
      host: comvita-mysupply.default.svc.cluster.local
      trafficPolicy:
        loadBalancer:
          consistentHash:
            httpHeaderName: version   # 1
    
## 3. Install ModHeader chrome extensio, then test. 
https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj 

# Shadowing — Virtual Services
Use when we want to test a change in production but not affect end-users, so we mirror the requests into a second instance that has the change and evaluate it.

## 1. Deploy two version

## 2. Specifying Subsets with Destination Rules

    kind: DestinationRule
    metadata:
      name: comvita-mysupply
    spec:
      host: comvita-mysupply.default.svc.cluster.local
      subsets:
      - name: v1        # 2
        labels:
          version: v1   # 3
      - name: v2
        labels:
          version: v2 

## 3. Route request to subset

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: comvita-mysupply
    spec:
      hosts:
        - comvita-mysupply.default.svc.cluster.local          
      http:
      - route:
        - destination:
            host: comvita-mysupply 
            subset: v1      
        mirror:             
          host: comvita-mysupply  
          subset: v2

# Canary Deployments
Canary Deployment is the process of rolling out a new version of an application to a small set of users, as a step to verify the absence of issues, and then with a higher assurance of quality release to the wider audience.

## 1. Deploy two version

## 2. Create VirtualService

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: comvita-mysupply
    spec:
      hosts:
        - comvita-mysupply.default.svc.cluster.local   
      http:
      - route: 
        - destination: 
            host: comvita-mysupply.default.svc.cluster.local
            subset: v1
          weight: 80       
        - destination: 
            host: comvita-mysupply.default.svc.cluster.local
            subset: v2
          weight: 20    

