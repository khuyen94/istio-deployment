Url: https://github.com/pantsel/konga/tree/master/charts/konga

## Install use Helm v2
    helm install .\kong\konga-admin\charts\konga\ --name my-konga --namespace kong

## Upgrade use Helm v2
    helm upgrade my-konga .\kong\konga-admin\charts\konga\ --namespace kong