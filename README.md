# Beder GitOps App-of-Apps

This repository contains the App-of-Apps pattern for managing multiple OpenShift clusters using ArgoCD. It uses a Helm chart to dynamically generate ArgoCD AppProjects and ApplicationSets for each cluster.

## Repository Structure

```
beder-gitops-aoa/
├── README.md
├── app-of-apps-application.yaml    # Root ArgoCD Application
└── app-of-apps/                    # Helm chart
    ├── Chart.yaml                  # Helm chart definition
    ├── values.yaml                 # Cluster configurations
    └── templates/
        ├── appproject.yaml         # Generates AppProjects
        └── applicationset.yaml     # Generates ApplicationSets
```

## How It Works

1. **Root Application**: `app-of-apps-application.yaml` deploys the Helm chart from `app-of-apps/`
2. **Helm Chart**: Reads `values.yaml` and generates:
   - **AppProjects** (sync wave 0) - ArgoCD projects for each cluster
   - **ApplicationSets** (sync wave 1) - Discovers and deploys applications from cluster repositories
3. **ApplicationSets**: Each ApplicationSet scans its cluster's Git repository and creates ArgoCD Applications for discovered directories

## Adding a New Cluster

To add a new cluster, update `app-of-apps/values.yaml`:

1. Add an AppProject entry:
```yaml
appprojects:
  - name: cluster3
    syncWave: "0"
```

2. Add a cluster configuration:
```yaml
clusters:
  - name: cluster3
    repoURL: https://github.com/yakovbeder/cluster3.git
    targetRevision: master
    project: cluster3
    destination: cluster3
    syncWave: "1"
```

## Configuration Fields

- `name`: Cluster identifier
- `repoURL`: Git repository URL containing cluster manifests
- `targetRevision`: Branch or tag (e.g., `master`, `main`)
- `project`: ArgoCD project name (should match cluster name)
- `destination`: Cluster name as registered in ArgoCD
- `syncWave`: Deployment order (AppProjects: "0", ApplicationSets: "1")

## Deployment

Deploy the root Application in ArgoCD:

```bash
oc apply -f app-of-apps-application.yaml
```

ArgoCD will automatically:
1. Deploy the Helm chart
2. Generate AppProjects (wave 0)
3. Generate ApplicationSets (wave 1)
4. ApplicationSets will discover and deploy applications from cluster repositories

