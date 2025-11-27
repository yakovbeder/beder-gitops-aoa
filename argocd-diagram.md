# ArgoCD Application Structure Diagram

## Application Hierarchy (Mermaid)

```mermaid
graph TD
    Root[app-of-apps<br/>Root Application] --> Wave0[Wave 0: AppProjects]
    Root --> Wave1[Wave 1: Global ApplicationSet]
    Root --> Wave2[Wave 2: Cluster ApplicationSets]
    
    Wave0 --> Proj1[cluster1-project]
    Wave0 --> Proj2[cluster2-project]
    
    Wave1 --> GlobalAS[global ApplicationSet]
    GlobalAS --> GlobalApp1[cluster1-htpasswd]
    GlobalAS --> GlobalApp2[cluster1-certificates]
    GlobalAS --> GlobalApp3[cluster2-htpasswd]
    GlobalAS --> GlobalApp4[cluster2-certificates]
    
    Wave2 --> Cluster1AS[cluster1 ApplicationSet]
    Wave2 --> Cluster2AS[cluster2 ApplicationSet]
    
    Cluster1AS --> App1[cluster1-cluster-scope]
    Cluster1AS --> App2[cluster1-operators]
    Cluster1AS --> App3[cluster1-etcd-backup]
    Cluster1AS --> App4[cluster1-machineconfig]
    Cluster1AS --> App5[cluster1-garbagecollection]
    
    Cluster2AS --> App6[cluster2-cluster-scope]
    Cluster2AS --> App7[cluster2-operators]
    Cluster2AS --> App8[cluster2-etcd-backup]
    Cluster2AS --> App9[cluster2-machineconfig]
    Cluster2AS --> App10[cluster2-garbagecollection]
    
    style Root fill:#e1f5ff
    style Wave0 fill:#fff4e1
    style Wave1 fill:#e8f5e9
    style Wave2 fill:#fce4ec
    style GlobalAS fill:#c8e6c9
    style Cluster1AS fill:#f8bbd0
    style Cluster2AS fill:#f8bbd0
```

## Sync Wave Flow

```mermaid
sequenceDiagram
    participant ArgoCD
    participant Wave0 as Wave 0<br/>AppProjects
    participant Wave1 as Wave 1<br/>Global Apps
    participant Wave2 as Wave 2<br/>Cluster Apps
    participant Cluster1
    participant Cluster2
    
    ArgoCD->>Wave0: Deploy AppProjects
    Wave0->>Cluster1: Create cluster1-project
    Wave0->>Cluster2: Create cluster2-project
    
    ArgoCD->>Wave1: Deploy Global ApplicationSet
    Wave1->>Cluster1: Deploy global configs<br/>(htpasswd, certificates)
    Wave1->>Cluster2: Deploy global configs<br/>(htpasswd, certificates)
    
    ArgoCD->>Wave2: Deploy Cluster ApplicationSets
    Wave2->>Cluster1: Deploy cluster1 apps<br/>(operators, etcd-backup, etc.)
    Wave2->>Cluster2: Deploy cluster2 apps<br/>(operators, etcd-backup, etc.)
```

## Repository Structure Flow

```mermaid
graph LR
    subgraph "beder-gitops-aoa repo"
        AOA[app-of-apps/] --> Helm[Helm Chart]
        Global[global-configs/] --> GlobalAS[Global ApplicationSet]
    end
    
    subgraph "cluster1 repo"
        C1[cluster1/] --> C1AS[cluster1 ApplicationSet]
        C1 --> C1Dir1[cluster-scope/]
        C1 --> C1Dir2[operators/]
        C1 --> C1Dir3[etcd-backup/]
    end
    
    subgraph "cluster2 repo"
        C2[cluster2/] --> C2AS[cluster2 ApplicationSet]
        C2 --> C2Dir1[cluster-scope/]
        C2 --> C2Dir2[operators/]
        C2 --> C2Dir3[etcd-backup/]
    end
    
    Helm --> Proj1[AppProject: cluster1]
    Helm --> Proj2[AppProject: cluster2]
    Helm --> GlobalAS
    Helm --> C1AS
    Helm --> C2AS
    
    GlobalAS --> GC1[cluster1-htpasswd]
    GlobalAS --> GC2[cluster2-htpasswd]
    
    C1AS --> App1[cluster1-cluster-scope]
    C1AS --> App2[cluster1-operators]
    
    C2AS --> App3[cluster2-cluster-scope]
    C2AS --> App4[cluster2-operators]
    
    style AOA fill:#e1f5ff
    style Global fill:#c8e6c9
    style C1 fill:#f8bbd0
    style C2 fill:#f8bbd0
```

## ApplicationSet Matrix Generator

```mermaid
graph TD
    Matrix[Matrix Generator] --> List[List Generator<br/>Clusters]
    Matrix --> Git[Git Generator<br/>Directories]
    
    List --> E1[cluster: cluster1<br/>destination: cluster1]
    List --> E2[cluster: cluster2<br/>destination: cluster2]
    
    Git --> D1[path: global-configs/htpasswd]
    Git --> D2[path: global-configs/certificates]
    
    E1 --> T1[Template Application]
    E2 --> T1
    D1 --> T1
    D2 --> T1
    
    T1 --> App1[cluster1-htpasswd]
    T1 --> App2[cluster1-certificates]
    T1 --> App3[cluster2-htpasswd]
    T1 --> App4[cluster2-certificates]
    
    style Matrix fill:#fff4e1
    style T1 fill:#e8f5e9
```

