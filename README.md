# CI/CD azure devops alternative

## Features zu untersuchen
- Runtime parameter mit values und freitext input feld
- pipeline templating
- build agent setup
- pipeline trigger
- hashicorp vault/azure keyvault integration
- deployment
## UseCase:
- Bauen und deployen einer app auf einem bestimmten namespace
- Bauen und pushen des docker images, deployen in einer zweiten pipeline
- Laden von secret von hashi corp vault
- Name des deployments soll mit einem freitext feld individuell angepasst werdn
## Technologien:
- GoCD
- ArgoCD
=> Jenkins + job builder plugin vs groovy pipeline
=> Jenkins + ArgoCD(Jenkins zum bauen, ArgoCD zum deployen)
- Concourse + ArgoCD(Concourse bauen, ArgoCD zum Deployen)
## Goal:
- Cop presentation
- Blog post
## Lokal:
- Kind/minikube für ci deployment
- Worker auch auf kind/minikube deployed
- Code auf github
- pushed public dockerhub
- HashiVault auf kind deployen
- Clustr deployment auf selben cluster