# platform-gitops

GitOps source of truth for multi-tenant application deployments.

## Repository layout

```text
bootstrap/
  argocd-app-of-apps.yaml
  applicationset-user-projects.yaml
apps/
  <workspaceNamespace>/
    <userId>/
      namespace.yaml
      <projectName>/
        Chart.yaml
        values.yaml
        templates/
          deployment.yaml
          service.yaml
          ingress.yaml
          hpa.yaml
```

## Deployment model

- Jenkins writes app release data to `apps/<workspaceNamespace>/<userId>/<projectName>/values.yaml`
- Jenkins writes the runtime env payload and immutable release data into `values.yaml`
- `image.tag` and `envJson` are updated for each deployment / rollback
- ArgoCD ApplicationSet discovers each app folder automatically (legacy and workspace-aware paths)
- Auto-sync is enabled with prune + self-heal
- For initial reset/bootstrapping, set ApplicationSet `prune: false` until first apps are created.
- App namespace is the workspace namespace created during onboarding:
  - `namespace = ns-<username>-<shortId>`

## Bootstrap

```bash
kubectl apply -n argocd -f bootstrap/argocd-app-of-apps.yaml
```

## Notes

- Do not store registry credentials directly in Git.
- Use Sealed Secrets or External Secrets for `registry-secret` in tenant namespaces.
