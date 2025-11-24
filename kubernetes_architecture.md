# Kubernetes (K8s) Introduction & Architecture

## 1. What is Kubernetes?
**Kubernetes (K8s)** is an open-source container orchestration platform. It acts as the "operating system" for your data center, providing infrastructure automation with self-healing and declarative state management.

**Core Responsibilities:**
*   Deploying and scaling containers.
*   Self-healing (restarting failed containers).
*   Networking and service discovery.
*   Storage management.
*   Rolling updates/rollbacks.

## 2. Architecture Overview
Kubernetes uses a **Master-Worker** architecture:
*   **Control Plane (Master)**: The "Brain" – decides what runs where.
*   **Data Plane (Worker Nodes)**: The "Muscles" – runs the actual applications.

### High-Level Layout
```mermaid
graph TD
    subgraph Control_Plane [Control Plane (Brain)]
        API[API Server]
        ETCD[etcd]
        SCH[Scheduler]
        CM[Controller Manager]
        CCM[Cloud Controller Mgr]
    end
    
    subgraph Worker_Nodes [Data Plane (Muscles)]
        subgraph Node1
            K1[Kubelet]
            P1[Kube-Proxy]
            R1[Runtime]
        end
        subgraph Node2
            K2[Kubelet]
            P2[Kube-Proxy]
            R2[Runtime]
        end
    end

    API --> ETCD
    API --> SCH
    API --> CM
    API --> CCM
    API --> K1
    API --> K2
```

## 3. Control Plane Components (The Brain)
| Component | Description |
| :--- | :--- |
| **kube-apiserver** | **The Front Door**. Validates requests, authenticates users, and updates state in etcd. All CLI (`kubectl`) calls go here. |
| **etcd** | **The Source of Truth**. Distributed key-value store for cluster data (config, secrets, state). Backups are critical. |
| **kube-scheduler** | **The Matchmaker**. Decides which node a pod should run on based on resources, taints, affinity, etc. |
| **kube-controller-manager** | **The Enforcer**. A loop that constantly checks if *Actual State* == *Desired State* (e.g., Node, Deployment, ReplicaSet controllers). |
| **cloud-controller-manager** | **The Cloud Integrator**. Manages cloud-specific resources like Load Balancers and Cloud Disks (AWS, Azure, GCP). |

## 4. Data Plane Components (The Muscle)
Runs on every **Worker Node**.

| Component | Description |
| :--- | :--- |
| **kubelet** | **Node Agent**. Talks to the API server, ensures containers are running/healthy, and pulls images. |
| **kube-proxy** | **Network Proxy**. Maintains network rules (iptables/ipvs) to allow service communication and load balancing. |
| **Container Runtime** | **The Engine**. Software that runs containers (e.g., containerd, CRI-O, Docker). Uses **CRI** (Container Runtime Interface). |

## 5. Additional Services
*   **CNI Plugin (Networking)**: Handles pod IP assignment and routing (e.g., Calico, Flannel, AWS VPC CNI).
*   **CSI Plugin (Storage)**: Handles persistent storage volumes (e.g., EBS, NFS, Ceph).

## 6. Workflow Example: `kubectl apply -f nginx.yaml`
1.  **User** sends command to **API Server**.
2.  **API Server** stores desired state in **etcd**.
3.  **Scheduler** detects pending pod, selects a node, and updates API server.
4.  **Kubelet** on selected node sees assignment, pulls image, and starts container.
5.  **Kube-proxy** updates network rules for traffic.
6.  **Controller Manager** ensures the correct number of replicas exist.
