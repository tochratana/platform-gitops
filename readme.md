# platform-gitops

GitOps state store for yourplatform.com deployments.

## How it works

- Jenkins writes to `apps/<app-name>/` after every successful build
- ArgoCD watches this repo and syncs changes to the Kubernetes cluster automatically
- Never edit files in `apps/` manually — let Jenkins manage them

## Bootstrap (run once)

kubectl apply -f bootstrap/namespace.yaml
kubectl apply -n argocd -f bootstrap/argocd-app-of-apps.yaml

## Adding a registry secret

kubectl create secret docker-registry registry-secret \
  --docker-server=registry.yourplatform.com \
  --docker-username=YOUR_USER \
  --docker-password=YOUR_PASS \
  --namespace=apps

## Repo rules

- `bootstrap/`  → applied manually once, never touched again
- `apps/`       → written by Jenkins CI only, never edit by hand
```

---

## The full deploy lifecycle in one picture
```
User clicks Deploy
      │
      ▼
Spring Boot backend
  • saves project (status = BUILDING)
  • calls Jenkins API
      │
      ▼
Jenkins pipeline
  • clones user repo
  • builds app + Docker image
  • pushes image to registry
  • runs update-gitops.sh
      │
      ▼
platform-gitops / apps / <app-name> /
  • deployment.yaml  ← image tag updated here
  • service.yaml
  • ingress.yaml
      │
      ▼
ArgoCD detects git push (polls every 3 min or webhook)
  • syncs deployment.yaml → Kubernetes
  • Kubernetes pulls new image, rolls out pod
      │
      ▼
https://<app-name>.yourplatform.com  ← live