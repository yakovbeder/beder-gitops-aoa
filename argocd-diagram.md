# ArgoCD Application Structure Diagram

## Application Hierarchy

```mermaid
flowchart TB
    subgraph root[" "]
        direction TB
        Root([🚀 app-of-apps<br/>Root Application])
    end

    subgraph wave0[" 🏗️ Wave 0: AppProjects "]
        Proj1[📁 in-cluster-project]
    end

    subgraph wave1[" 🌍 Wave 1: Global ApplicationSet "]
        GlobalAS{{🔄 global-applicationset}}
        subgraph globalApps[" Global Apps → All Clusters "]
            GlobalApp1[🔐 htpasswd]
            GlobalApp2[📜 certificates]
            GlobalApp3[📦 resources]
        end
    end

    subgraph wave2[" 🎯 Wave 2: Cluster ApplicationSets "]
        subgraph cluster1[" 🖥️ in-cluster "]
            Cluster1AS{{🔄 in-cluster-applicationset}}
            App1[⚙️ cluster-scope]
            App2[📦 operators]
            App3[💾 etcd-backup]
            App4[🔧 machineconfig]
            App5[🗑️ garbagecollection]
        end
    end

    Root --> wave0
    Root --> wave1
    Root --> wave2

    GlobalAS --> globalApps

    Cluster1AS --> App1
    Cluster1AS --> App2
    Cluster1AS --> App3
    Cluster1AS --> App4
    Cluster1AS --> App5

    %% Styling
    style root fill:#1a73e8,stroke:#1557b0,color:#fff
    style Root fill:#1a73e8,stroke:#1557b0,color:#fff
    
    style wave0 fill:#ff9800,stroke:#e65100,color:#000
    style Proj1 fill:#ffe0b2,stroke:#ff9800
    
    style wave1 fill:#4caf50,stroke:#2e7d32,color:#fff
    style GlobalAS fill:#81c784,stroke:#4caf50
    style globalApps fill:#c8e6c9,stroke:#4caf50
    style GlobalApp1 fill:#e8f5e9,stroke:#81c784
    style GlobalApp2 fill:#e8f5e9,stroke:#81c784
    style GlobalApp3 fill:#e8f5e9,stroke:#81c784
    
    style wave2 fill:#e91e63,stroke:#ad1457,color:#fff
    style cluster1 fill:#fce4ec,stroke:#e91e63
    style Cluster1AS fill:#f48fb1,stroke:#e91e63
    style App1 fill:#fff,stroke:#f48fb1
    style App2 fill:#fff,stroke:#f48fb1
    style App3 fill:#fff,stroke:#f48fb1
    style App4 fill:#fff,stroke:#f48fb1
    style App5 fill:#fff,stroke:#f48fb1
```

## Sync Wave Flow

```mermaid
sequenceDiagram
    autonumber
    
    box rgb(26, 115, 232) ArgoCD Control Plane
        participant ArgoCD as 🎮 ArgoCD
    end
    
    box rgb(255, 152, 0) Wave 0
        participant Wave0 as 🏗️ AppProjects
    end
    
    box rgb(76, 175, 80) Wave 1
        participant Wave1 as 🌍 Global Apps
    end
    
    box rgb(233, 30, 99) Wave 2
        participant Wave2 as 🎯 Cluster Apps
    end
    
    box rgb(96, 125, 139) Target Clusters
        participant InCluster as 🖥️ in-cluster
    end

    Note over ArgoCD: Helm chart generates<br/>all resources

    ArgoCD->>+Wave0: 1️⃣ Deploy AppProjects
    Wave0->>InCluster: Create in-cluster-project
    Wave0-->>-ArgoCD: ✅ Projects ready

    ArgoCD->>+Wave1: 2️⃣ Deploy Global ApplicationSet
    Wave1->>InCluster: Deploy htpasswd
    Wave1->>InCluster: Deploy certificates
    Wave1->>InCluster: Deploy resources
    Wave1-->>-ArgoCD: ✅ Global configs deployed

    ArgoCD->>+Wave2: 3️⃣ Deploy Cluster ApplicationSets
    Wave2->>InCluster: Discover & deploy apps<br/>(cluster-scope, operators,<br/>etcd-backup, machineconfig,<br/>garbagecollection)
    Wave2-->>-ArgoCD: ✅ Cluster apps deployed

    Note over ArgoCD,InCluster: 🎉 All applications synced!
```

## Repository Relationship

```mermaid
flowchart LR
    subgraph github[" ☁️ GitHub "]
        direction TB
        subgraph aoa[" 📂 beder-gitops-aoa "]
            AOARoot[app-of-apps-application.yaml]
            HelmChart[app-of-apps/<br/>Helm Chart]
            GlobalConfigs[global-configs/<br/>htpasswd, certificates, resources]
        end
        
        subgraph clusterRepos[" 📂 Cluster Repositories "]
            InClusterRepo[in-cluster.git<br/>cluster-scope, operators, etc.]
        end
    end

    subgraph argocd[" 🎮 ArgoCD "]
        RootApp([Root Application])
        AppSets([ApplicationSets])
        Apps([Applications])
    end

    subgraph ocp[" 🔴 OpenShift Clusters "]
        InClusterOCP[in-cluster]
    end

    AOARoot -->|deploys| RootApp
    HelmChart -->|generates| AppSets
    GlobalConfigs -->|source for| Apps
    InClusterRepo -->|source for| Apps
    
    RootApp -->|creates| AppSets
    AppSets -->|creates| Apps
    Apps -->|syncs to| InClusterOCP

    style github fill:#24292e,stroke:#1b1f23,color:#fff
    style aoa fill:#0366d6,stroke:#0256b9,color:#fff
    style clusterRepos fill:#28a745,stroke:#22863a,color:#fff
    style argocd fill:#e91e63,stroke:#ad1457,color:#fff
    style ocp fill:#ee0000,stroke:#cc0000,color:#fff
    
    style AOARoot fill:#58a6ff,stroke:#0366d6
    style HelmChart fill:#58a6ff,stroke:#0366d6
    style GlobalConfigs fill:#58a6ff,stroke:#0366d6
    style InClusterRepo fill:#7ee787,stroke:#28a745
    
    style RootApp fill:#f48fb1,stroke:#e91e63
    style AppSets fill:#f48fb1,stroke:#e91e63
    style Apps fill:#f48fb1,stroke:#e91e63
    
    style InClusterOCP fill:#ff6b6b,stroke:#ee0000
```

## Legend

| Symbol | Meaning |
|--------|---------|
| 🚀 | Root Application |
| 🏗️ | AppProjects (Wave 0) |
| 🌍 | Global Configurations (Wave 1) |
| 🎯 | Cluster-specific Apps (Wave 2) |
| 🔄 | ApplicationSet |
| 📁 | AppProject |
| 🖥️ | Cluster |
| ⚙️ | Configuration |
| 📦 | Operators/Resources |
| 🔐 | Authentication |
| 📜 | Certificates |
| 💾 | Backup |
| 🔧 | Machine Config |
| 🗑️ | Garbage Collection |

