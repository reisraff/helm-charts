# GitOps Deployment Arch

This document has instructions to up an automated GitOps deployment architecture using the following technologies:

- [Kubernets](https://kubernetes.io/)
    - K8S Cluster Locally
        - [Minukube](https://minikube.sigs.k8s.io/docs/start/)
    - K8S Cluster in Cloud
        - [EKS](https://aws.amazon.com/eks/)
        - [GKE](https://cloud.google.com/kubernetes-engine)
- [Kustomize](https://kustomize.io/)
- [ArgoCD](https://argoproj.github.io/)
- [HELM](https://helm.sh/)
- [CrossPlane](https://www.crossplane.io/) **(@todo)**

## Setup a kubernetes cluster

### K8S Cluster Locally

With minikube we have to add the ingress controller addon

```bash
minikube addons enable ingress
```

Then start
```bash
minikube start
```

### K8S Cluster in Cloud

#### GKE
Create a cluster in GKE

@see https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress

@see https://cloud.google.com/kubernetes-engine/docs/concepts/ingress

```bash
gcloud container clusters create $K_CLUSTER_ARGOCD_NAME \
    --zone=$ZONE \
    --total-min-nodes=1 \
    --total-max-nodes=$MAX_NODES \
    --create-subnetwork name=$VPC_NAME \
    --no-enable-master-authorized-networks \
    --enable-ip-alias \
    --enable-private-nodes \
    --master-ipv4-cidr $IPV4_CIDR
```

Get authenticated with the cluster

```bash
gcloud container clusters get-credentials {cluster-name} \
  --region={compute-region}
```
#### EKS

@TODO

## ArgoCD

Create a argocd namespace and install argocd manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Port forwarding to test
**IMPORTANT**: this is just for test, we have to expose to the internet (or over VPN) when it is in prod

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Update admin password

Get the current password for the default user (admin)

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

Login via CLI

```bash
argocd login localhost:8080
```

Update the password

```bash
argocd account update-password
```

## External Secrets

@see [deployments/external-secrets/README.md](https://github.com/reisraff/helm-charts/blob/main/deployments/external-secrets/README.md)

## Helm Charts

@see [README.md](https://github.com/reisraff/helm-charts/blob/main/README.md)

## Automated Deploys

@see [deployments/apps/README.md](https://github.com/reisraff/helm-charts/blob/main/deployments/apps/README.md)

## Manually Helm deployment

```bash
# Add your helm charts repo
helm repo add reisraff-apps https://reisraff.github.io/helm-charts/

# Update repo
helm repo update

# Update app charts dependencies
helm dependency update prod/simple-app/

# Rebuild app charts dependencies
helm dependency build prod/simple-app/

# Check if the artifacts are ok
helm template prod/simple-app/

# Install the app
helm install -f prod/simple-app/values.yaml simple-app prod/simple-app/

# Upgrade the app
helm upgrade -i -f prod/simple-app/values.yaml simple-app prod/simple-app/
```

## Simple app template

A simple App Template can be found [here](https://github.com/reisraff/apps/tree/c403638ba1a6495274b535537d2f49a486bbb30b)

May the the actual chart version does not work anymore with this simple app

## Crossplane

@todo (nice to have)
