# ArgoCD App of Apps Setup

This repository contains an ArgoCD "App of Apps" pattern configuration.

## Structure

```
argo/
├── app-of-apps.yaml    # Root application that manages all child apps
├── apps/               # Directory containing child application definitions
│   ├── app1.yaml
│   ├── app2.yaml
│   └── kustomization.yaml
└── README.md
```

## App of Apps Pattern

The "App of Apps" pattern allows you to manage multiple ArgoCD applications through a single parent application. This provides:

- Centralized management of multiple applications
- Automated synchronization
- Self-healing capabilities
- Easy scaling of application deployments

## Setup Instructions

### 1. Update Repository URLs

Before applying, update the `repoURL` fields in all YAML files to point to your Git repository:

```yaml
source:
  repoURL: https://github.com/your-org/your-repo.git  # Update this
```

### 2. Apply the Root Application

Apply the root application to your ArgoCD instance:

```bash
kubectl apply -f app-of-apps.yaml
```

Or if you want to use ArgoCD CLI:

```bash
argocd app create -f app-of-apps.yaml
```

### 3. Verify

Check the status of the root application:

```bash
kubectl get applications -n argocd
argocd app get app-of-apps
```

## How It Works

1. **Root Application** (`app-of-apps.yaml`): This application watches the `apps/` directory and automatically creates/manages all child applications defined there.

2. **Child Applications** (`apps/*.yaml`): Each child application manages its own Kubernetes resources from a specific path in your Git repository.

3. **Automated Sync**: Both root and child applications have automated sync enabled with:
   - `prune: true` - Automatically removes resources that are no longer in Git
   - `selfHeal: true` - Automatically corrects any manual changes to match Git state

## Adding New Applications

To add a new application:

1. Create a new YAML file in the `apps/` directory (e.g., `apps/app3.yaml`)
2. Add it to `apps/kustomization.yaml` if using Kustomize
3. Commit and push to your Git repository
4. ArgoCD will automatically detect and create the new application

## Example Child Application

Each child application follows this structure:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <app-name>
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <your-repo-url>
    targetRevision: HEAD
    path: manifests/<app-name>
  destination:
    server: https://kubernetes.default.svc
    namespace: <app-namespace>
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Customization

- **Sync Policy**: Modify `syncPolicy` to change sync behavior (manual, automated, etc.)
- **Projects**: Use ArgoCD projects for RBAC and resource restrictions
- **Helm/Kustomize**: Child applications can use Helm charts or Kustomize overlays
- **Multi-cluster**: Update `destination.server` to deploy to different clusters

## Troubleshooting

- Check application status: `argocd app get <app-name>`
- View sync history: `argocd app history <app-name>`
- Check logs: `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller`

