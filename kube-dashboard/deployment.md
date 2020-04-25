# How to deploy Kubernetes Dashboard

## 1. Deploy dashboard using `kubectl`
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

## 2. Create Service Account
    kubectl apply -f .\kube-dashboard\service-account.yaml

## 3. Create ClusterRoleBinding
    kubectl apply -f .\kube-dashboard\cluster-role-binding.yaml

## 4. Get Bearer Token
Now we need to find token we can use to log in. Execute following command:
For Bash:

    kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

For Powershell:

    kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | sls admin-user | ForEach-Object { $_ -Split '\s+' } | Select -First 1)

## 5. Start dashboard proxy
    kubectl proxy

Then input the Bearer Token