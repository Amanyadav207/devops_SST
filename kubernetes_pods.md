# Kubernetes Pods: Deep Dive

## 1. Quick Summary
A **Pod** is the smallest deployable unit in Kubernetes, consisting of one or more co-scheduled containers sharing network namespace (IP), IPC, and volumes.

## 2. Architecture & Roles
*   **Control Plane**:
    *   **API Server**: Single entry point (REST). Validates, authenticates, writes to etcd.
    *   **etcd**: Source of truth (consistent key-value store).
    *   **Scheduler**: Assigns unscheduled Pods to Nodes (`spec.nodeName`).
    *   **Controller Manager**: Reconciles state (ReplicaSets, Deployments, etc.).
*   **Data Plane (Node)**:
    *   **kubelet**: Node agent. Watches API, starts containers via **CRI**, monitors health.
    *   **Container Runtime**: Pulls images, runs containers (containerd, CRI-O).
    *   **kube-proxy**: Manages network rules (iptables/ipvs) for Services.

## 3. Pod Creation Flow (The Lifecycle)
1.  **Request**: `kubectl apply -f pod.yaml` → **API Server**.
2.  **Auth & Webhooks**:
    *   **AuthN/AuthZ**: Checks identity and permissions.
    *   **Mutating Webhooks**: May modify pod (e.g., inject sidecars).
    *   **Validating Webhooks**: Rejects invalid requests (e.g., security policy).
3.  **Persist**: API Server writes Pod to **etcd**.
4.  **Schedule**:
    *   **Scheduler** detects pending pod.
    *   Filters nodes (resources, taints) & Scores them.
    *   Writes binding (`spec.nodeName` set to chosen node).
5.  **Execution (Node)**:
    *   **Kubelet** sees assignment.
    *   Calls **CNI** for IP address.
    *   Calls **CRI** (Runtime) to pull images and start containers.
6.  **Status**: Kubelet updates `status` (Pending → Running).
7.  **Service**: **kube-proxy** and Endpoints controller add Pod IP to Service endpoints when **Readiness Probe** passes.

## 4. Critical Concepts
*   **etcd Storage**: Stores `spec` (desired) and `status` (actual). Uses `resourceVersion` for optimistic concurrency (prevents race conditions).
*   **Networking**:
    *   **CNI**: Assigns Pod IP.
    *   **Pause Container**: Holds the network namespace.
*   **Probes**:
    *   **Liveness**: Restarts container if it fails.
    *   **Readiness**: Controls traffic flow (Service only sends traffic if Ready).

## 5. Debugging Cheatsheet
| Issue | Cause | Check Command |
| :--- | :--- | :--- |
| **Pending** | No resources, Taints/Tolerations mismatch. | `kubectl describe pod <pod>` |
| **ImagePullBackOff** | Wrong image name, missing secret, auth fail. | `kubectl get pod` (look at status) |
| **CrashLoopBackOff** | App crashing, fails liveness probe. | `kubectl logs <pod>` |
| **ContainerCreating** | CNI net config fail, Volume mount fail. | `kubectl describe pod`, Node logs |
| **Access Denied** | RBAC permission issues. | Check ServiceAccount/RoleBinding |

## 6. Best Practices
*   **Never** create raw Pods (use Deployments/StatefulSets).
*   **Always** define Resource Requests/Limits.
*   **Use Probes** (Readiness & Liveness).
*   **Security**: Use non-root users, implement Pod Security Standards.
