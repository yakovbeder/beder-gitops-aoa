# Beder GitOps App-of-Apps

This repository contains the App-of-Apps pattern for managing multiple OpenShift clusters with ArgoCD. The `app-of-apps/` Helm chart reads `values.yaml` and generates AppProjects plus ApplicationSets for both global and cluster-specific content.

## Repository Structure

```
beder-gitops-aoa/
├── README.md
├── app-of-apps-application.yaml    # Root ArgoCD Application
├── app-of-apps/                    # Helm chart
│   ├── Chart.yaml                  # Helm chart definition
│   ├── values.yaml                 # Global and cluster configuration
│   └── templates/
│       ├── appproject.yaml              # Generates AppProjects from values
│       ├── applicationset.yaml         # Generates per-cluster ApplicationSets
│       └── applicationset-global.yaml  # Generates the global ApplicationSet
└── global-configs/                 # Shared configuration directories
    ├── htpasswd/                   # Example: authentication configuration
    ├── certificates/               # Example: shared certificates
    ├── resources/                  # Shared namespace-scoped resources
    │   └── namespaces/             # Namespace definitions
    └── ...                         # Other shared configuration directories
```

## How It Works

1. `app-of-apps-application.yaml` deploys the Helm chart from `app-of-apps/`.
2. The Helm chart reads `app-of-apps/values.yaml` and generates:
   - AppProjects (sync wave 0): one global AppProject when `global.enabled` is true, plus one AppProject per cluster entry.
   - Global ApplicationSet (sync wave 1): discovers and deploys shared content from `global-configs/` to all configured clusters.
   - Cluster ApplicationSets (sync wave 2): discovers and deploys top-level directories from each cluster repository.
3. The generated ApplicationSets create ArgoCD Applications automatically:
   - Global applications are named `global-{cluster}-{directory}`.
   - Cluster-specific applications are named `{cluster}-{directory}`.

## Cluster Repository Structure

Each cluster repository (for example, `cluster1.git`) is expected to contain top-level directories that ArgoCD can discover automatically:

```
cluster1/
├── cluster-scope/     # Cluster-scoped resources
├── operators/         # Operator configuration
├── etcd-backup/       # ETCD backup configuration
├── machineconfig/     # MachineConfig resources
└── garbagecollection/ # Garbage collection configuration
```

The cluster ApplicationSet creates one ArgoCD Application per top-level directory and recursively deploys manifests from that directory.

## Global Configuration

Global configurations are shared across all configured clusters and are stored under `global-configs/` in this repository.

### Configuration

The `global` section in `app-of-apps/values.yaml` controls the shared ApplicationSet:

```yaml
global:
  enabled: true
  repoURL: git@github.com:yakovbeder/beder-gitops-aoa.git
  targetRevision: v2.0
  project: global
  path: "global-configs"
  syncWave: "1"
```

Set `enabled: false` to disable the global ApplicationSet without removing the configuration.

The global ApplicationSet will:
- Discover directories directly under `global-configs/`
- Deploy each discovered directory to every cluster listed in `clusters`
- Create applications named `global-{cluster}-{directory}`

### Global Configuration Directory Structure

The `global-configs/` directory can contain shared folders such as:

```
global-configs/
├── htpasswd/         # Example: authentication configuration
├── certificates/     # Example: shared certificates
├── resources/        # Shared namespace resources
│   └── namespaces/   # Namespace definitions
└── other-config/     # Example: additional shared configuration
```

## Adding a New Cluster

To add a new cluster, update `app-of-apps/values.yaml` with a new entry under `clusters`:

```yaml
clusters:
  - name: cluster3
    repoURL: git@github.com:yakovbeder/cluster3.git
    targetRevision: v2.0
    project: cluster3
    destination: cluster3
    syncWave: "2"
```

The chart will automatically create:
- An AppProject for the cluster
- A cluster-specific ApplicationSet for that repository
- Global applications for the cluster when `global.enabled` is true

## Configuration Fields

### Global
- `enabled`: Enables or disables the global ApplicationSet
- `repoURL`: Repository URL containing the shared configuration
- `targetRevision`: Branch, tag, or revision to deploy
- `project`: ArgoCD project name for global applications
- `path`: Root directory containing shared configuration folders
- `syncWave`: Sync wave for the generated global ApplicationSet

### Clusters
- `name`: Cluster identifier used in generated application names
- `repoURL`: Repository URL containing cluster manifests
- `targetRevision`: Branch, tag, or revision to deploy
- `project`: ArgoCD project name generated for the cluster
- `destination`: Cluster name as registered in ArgoCD
- `syncWave`: Sync wave for the generated cluster ApplicationSet

### Sync Wave Order
1. Wave 0: AppProjects are created.
2. Wave 1: Global configurations are deployed to all configured clusters.
3. Wave 2: Cluster-specific configurations are deployed.

## Deployment

Deploy the root Application in ArgoCD:

```bash
oc apply -f app-of-apps-application.yaml
```

ArgoCD will then:
1. Deploy the Helm chart.
2. Generate AppProjects from the `global` and `clusters` configuration.
3. Generate the global ApplicationSet when enabled.
4. Generate the cluster-specific ApplicationSets.

