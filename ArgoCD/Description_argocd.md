# ArgoCD on Kubernetes

#   workflow
following the [getting started](https://argoproj.github.io/argo-cd/getting_started/)

preferably also via done helm on minikube via
```bash
helm repo add argo https://argoproj.github.io/argo-helm
"argo" has been added to your repositories

helm install --name devops-tools argo/argo-cd
```
or via install.yml provided in argocd github repo
```bash
kubectl apply -n devops-tools -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
