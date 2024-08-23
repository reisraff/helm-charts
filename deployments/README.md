# README

## Apply the `apps` kustomization

This is the application/kustomization that will look into the `apps` repo and will deploy all apps over there

```bash
kubectl apply -k apps/
```

## Apply the `external-secrets`

```
kubectl apply -f external-secrets/external-secrets-{gcp|aws}.yaml
```
