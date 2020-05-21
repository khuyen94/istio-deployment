1. Create ConfigMap 

Go to "C:\lua\share\plugins". Then

        kubectl create configmap kong-plugin-oidc --from-file=.\plugins\oidc -n kong  
  
2. Access to kong proxy pod via dashboard portal

3. Run following command to install kong-oidc: `luarocks install comvita-kong-oidc`

