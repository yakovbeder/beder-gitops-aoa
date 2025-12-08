# ArgoCD Application Structure Diagram

## Application Hierarchy

```mermaid
flowchart TB
    Root([app-of-apps Root Application])

    subgraph wave0["Wave 0"]
        Proj1[in-cluster-project]
    end

    subgraph wave1["Wave 1"]
        GlobalAS{{global-applicationset}}
        GlobalApp1[htpasswd]
        GlobalApp2[certificates]
        GlobalApp3[resources]
    end

    subgraph wave2["Wave 2"]
        subgraph cluster1["in-cluster"]
            Cluster1AS{{in-cluster-applicationset}}
            App1[cluster-scope]
            App2[operators]
            App3[etcd-backup]
            App4[machineconfig]
            App5[garbagecollection]
        end
    end

    Root --> wave0
    Root --> wave1
    Root --> wave2

    GlobalAS --> GlobalApp1
    GlobalAS --> GlobalApp2
    GlobalAS --> GlobalApp3

    Cluster1AS --> App1
    Cluster1AS --> App2
    Cluster1AS --> App3
    Cluster1AS --> App4
    Cluster1AS --> App5

    style Root fill:#1a73e8,stroke:#1557b0,color:#fff
    
    style wave0 fill:#ff9800,stroke:#e65100,color:#000
    style Proj1 fill:#ffe0b2,stroke:#ff9800,color:#000
    
    style wave1 fill:#4caf50,stroke:#2e7d32,color:#000
    style GlobalAS fill:#a5d6a7,stroke:#4caf50,color:#000
    style GlobalApp1 fill:#e8f5e9,stroke:#81c784,color:#000
    style GlobalApp2 fill:#e8f5e9,stroke:#81c784,color:#000
    style GlobalApp3 fill:#e8f5e9,stroke:#81c784,color:#000
    
    style wave2 fill:#e91e63,stroke:#ad1457,color:#000
    style cluster1 fill:#fce4ec,stroke:#e91e63,color:#000
    style Cluster1AS fill:#f48fb1,stroke:#e91e63,color:#000
    style App1 fill:#fff,stroke:#f48fb1,color:#000
    style App2 fill:#fff,stroke:#f48fb1,color:#000
    style App3 fill:#fff,stroke:#f48fb1,color:#000
    style App4 fill:#fff,stroke:#f48fb1,color:#000
    style App5 fill:#fff,stroke:#f48fb1,color:#000
```

## Sync Wave Flow

```mermaid
sequenceDiagram
    autonumber
    
    participant ArgoCD
    participant Projects as AppProjects
    participant Global as Global Apps
    participant Cluster as Cluster Apps
    participant Target as in-cluster

    Note over ArgoCD: Helm chart generates all resources

    rect rgb(255, 243, 224)
        Note over ArgoCD,Projects: Wave 0
        ArgoCD->>Projects: Deploy AppProjects
        Projects->>Target: Create in-cluster-project
        Projects-->>ArgoCD: Done
    end

    rect rgb(232, 245, 233)
        Note over ArgoCD,Global: Wave 1
        ArgoCD->>Global: Deploy Global ApplicationSet
        Global->>Target: Deploy htpasswd
        Global->>Target: Deploy certificates
        Global->>Target: Deploy resources
        Global-->>ArgoCD: Done
    end

    rect rgb(252, 228, 236)
        Note over ArgoCD,Cluster: Wave 2
        ArgoCD->>Cluster: Deploy Cluster ApplicationSets
        Cluster->>Target: Deploy cluster-scope
        Cluster->>Target: Deploy operators
        Cluster->>Target: Deploy etcd-backup
        Cluster->>Target: Deploy machineconfig
        Cluster->>Target: Deploy garbagecollection
        Cluster-->>ArgoCD: Done
    end

    Note over ArgoCD,Target: All applications synced!
```

## Repository Relationship

```mermaid
flowchart LR
    subgraph github["GitHub"]
        subgraph aoa["beder-gitops-aoa"]
            AOARoot[app-of-apps-application.yaml]
            HelmChart[app-of-apps/]
            GlobalConfigs[global-configs/]
        end
        
        subgraph repos["Cluster Repos"]
            InClusterRepo[in-cluster.git]
        end
    end

    subgraph argocd["ArgoCD"]
        RootApp([Root App])
        AppSets([AppSets])
        Apps([Apps])
    end

    subgraph ocp["OpenShift"]
        InClusterOCP[in-cluster]
    end

    AOARoot --> RootApp
    HelmChart --> AppSets
    GlobalConfigs --> Apps
    InClusterRepo --> Apps
    
    RootApp --> AppSets
    AppSets --> Apps
    Apps --> InClusterOCP

    style github fill:#24292e,stroke:#1b1f23,color:#fff
    style aoa fill:#0366d6,stroke:#0256b9,color:#fff
    style repos fill:#28a745,stroke:#22863a,color:#fff
    style argocd fill:#e91e63,stroke:#ad1457,color:#fff
    style ocp fill:#ee0000,stroke:#cc0000,color:#fff
    
    style AOARoot fill:#58a6ff,stroke:#0366d6,color:#000
    style HelmChart fill:#58a6ff,stroke:#0366d6,color:#000
    style GlobalConfigs fill:#58a6ff,stroke:#0366d6,color:#000
    style InClusterRepo fill:#7ee787,stroke:#28a745,color:#000
    
    style RootApp fill:#f48fb1,stroke:#e91e63,color:#000
    style AppSets fill:#f48fb1,stroke:#e91e63,color:#000
    style Apps fill:#f48fb1,stroke:#e91e63,color:#000
    
    style InClusterOCP fill:#ff6b6b,stroke:#ee0000,color:#000
```

## Legend

| Symbol | Meaning |
|--------|---------|
| Wave 0 (Orange) | AppProjects |
| Wave 1 (Green) | Global Configurations |
| Wave 2 (Pink) | Cluster-specific Apps |
| Diamond shape | ApplicationSet |
| Stadium shape | Application |
| Rectangle | Resource/Config |

