# Beder GitOps App-of-Apps

This repository contains the App-of-Apps pattern for managing multiple OpenShift clusters using ArgoCD. It uses a Helm chart to dynamically generate ArgoCD AppProjects and ApplicationSets for each cluster.

## Repository Structure

```
beder-gitops-aoa/
├── README.md
├── app-of-apps-application.yaml    # Root ArgoCD Application
├── app-of-apps/                    # Helm chart
│   ├── Chart.yaml                  # Helm chart definition
│   ├── values.yaml                 # Cluster configurations
│   └── templates/
│       ├── appproject.yaml              # Generates AppProjects (one per cluster)
│       ├── applicationset.yaml         # Generates ApplicationSets (one per cluster)
│       └── applicationset-global.yaml  # Generates ApplicationSet (one for all clusters)
└── global-configs/                 # Global configurations directory
    ├── htpasswd/                   # Authentication configuration
    ├── certificates/               # Shared certificates
    └── ...                         # Other shared configurations
```

## How It Works

1. **Root Application**: `app-of-apps-application.yaml` deploys the Helm chart from `app-of-apps/`
2. **Helm Chart**: Reads `values.yaml` and generates:
   - **AppProjects** (sync wave 0) - ArgoCD projects for each cluster
   - **Global ApplicationSet** (sync wave 1) - Discovers and deploys global configurations to all clusters
   - **Cluster ApplicationSets** (sync wave 2) - Discovers and deploys applications from cluster repositories
3. **ApplicationSets**: 
   - **Global**: Scans the `global-configs/` directory in the same repository and deploys all directories to all clusters
   - **Cluster-specific**: Each ApplicationSet scans its cluster's Git repository and creates ArgoCD Applications for discovered directories

## Cluster Repository Structure

Each cluster repository (e.g., `cluster1.git`) should have the following structure:

```
cluster1/
├── cluster-scope/      # Cluster-scoped resources (ConfigMaps, IngressOperator config, etc.)
├── operators/          # Operator configurations (monitoring, logging, trident, etc.)
├── etcd-backup/        # ETCD backup configurations
├── machineconfig/      # MachineConfig resources
└── garbagecollection/   # Garbage collection configurations
```

The ApplicationSet will automatically discover each top-level directory and create an ArgoCD Application for it. Each Application will recursively deploy all manifests found within its directory.

## Global Configuration

Global configurations are shared across all clusters (e.g., htpasswd, certificates). They are stored in the `global-configs/` directory at the repository root and deployed to all clusters automatically.

### Configuration

In `app-of-apps/values.yaml`, add the `global` section:

```yaml
global:
  repoURL: https://github.com/yakovbeder/beder-gitops-aoa.git
  targetRevision: master
  project: default
  path: "global-configs"
  syncWave: "1"
```

The global ApplicationSet will:
- Discover all directories in the `global-configs/` directory of the same repository
- Deploy each directory to all clusters defined in `clusters`
- Create Applications named `{cluster-name}-{directory-name}`

### Global Configuration Directory Structure

The global configurations are stored in the `global-configs/` directory at the repository root:

```
global-configs/
├── htpasswd/          # Authentication configuration
├── certificates/      # Shared certificates
└── other-config/     # Other shared configurations
```

## Adding a New Cluster

To add a new cluster, update `app-of-apps/values.yaml`:

1. Add an AppProject entry:
```yaml
projects:
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
    syncWave: "2"
```

## Configuration Fields

### Projects
- `name`: Cluster identifier
- `syncWave`: Deployment order (always "0")

### Global
- `repoURL`: Git repository URL (same as app-of-apps repository)
- `targetRevision`: Branch or tag (e.g., `master`, `main`)
- `project`: ArgoCD project name (typically "default")
- `path`: Directory path containing global configurations (e.g., "global-configs")
- `syncWave`: Deployment order (always "1")

### Clusters
- `name`: Cluster identifier
- `repoURL`: Git repository URL containing cluster manifests
- `targetRevision`: Branch or tag (e.g., `master`, `main`)
- `project`: ArgoCD project name (should match cluster name)
- `destination`: Cluster name as registered in ArgoCD
- `syncWave`: Deployment order (always "2")

### Sync Wave Order
1. **Wave 0**: AppProjects are created
2. **Wave 1**: Global configurations are deployed to all clusters
3. **Wave 2**: Cluster-specific configurations are deployed

## Deployment

Deploy the root Application in ArgoCD:

```bash
oc apply -f app-of-apps-application.yaml
```

ArgoCD will automatically:
1. Deploy the Helm chart
2. Generate AppProjects (wave 0)
3. Generate Global ApplicationSet (wave 1) - deploys global configs to all clusters
4. Generate Cluster ApplicationSets (wave 2) - discover and deploy applications from cluster repositories

