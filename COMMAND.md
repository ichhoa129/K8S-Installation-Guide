# Kubernetes common command


## Namespace 
### Switch namespace
```sh
kubectl config set-context --current --namespace=<namespace>
```
### Create context
```sh
kubectl config set-context gce-dev --user=cluster-admin --namespace=dev
kubectl config use-context gce-dev
```

## Pod
### Get log
```sh
kubectl logs <pod> -n your-namespace
```