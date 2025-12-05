# Kubernetes Deployment: The Master Controller

## 1. Hierarchy Overview
*   **Pod**: Basic unit. Ephemeral. Dies with the Node.
*   **ReplicaSet (RS)**: Ensures `X` copies of a Pod are running. Handles scaling & healing.
*   **Deployment**: Manages ReplicaSets. Provides declarative updates, rollbacks, and versioning. **Use this for stateless apps.**

## 2. Deployment Features
*   **Rolling Updates**: Seamless zero-downtime upgrades.
*   **Rollbacks**: Revert to previous revisions instantly.
*   **Self-Healing**: Automatically replaces failed pods (via RS).
*   **Scalability**: Scale replicas up/down declaratively.

## 3. Deployment Workflow
1.  **Request**: `kubectl apply -f deploy.yaml` → API Server (validates & stores in etcd).
2.  **Controller**: Deployment Controller sees new/updated object.
    *   Creates a **new ReplicaSet** (if new template).
    *   Or updates existing RS.
3.  **Scaling**: ReplicaSet creates/deletes Pods to match desired state.
4.  **Scheduling & Execution**: Scheduler assigns nodes; Kubelet runs containers.

## 4. Rollout Strategies
| Strategy | Description | Use Case |
| :--- | :--- | :--- |
| **RollingUpdate** (Default) | Gradually replaces old pods with new ones (e.g., `maxSurge: 25%`, `maxUnavailable: 25%`). | Zero-downtime standard updates. |
| **Recreate** | Kills **all** old pods, *then* creates new ones. | Apps that can't run two versions simultaneously. |

## 5. Deep Internals (etcd & Revisions)
*   **Storage**: Deployments are stored in etcd as serialized **Protobuf/JSON** keys (e.g., `/registry/deployments/default/webapp`).
*   **Revisions**: Every update creates a **new ReplicaSet** with a unique revision annotation (`deployment.kubernetes.io/revision: "3"`).
*   **Rollback**: The Deployment simply starts pointing to an older ReplicaSet revision.
*   **Reconciliation**: The controller loop constantly compares "Desired State" (etcd) vs "Actual State" (Kubelets). Mismatches trigger immediate fixes (e.g., creating a pod if one vanishes).

## 6. Essential Commands
```bash
# Update Image (Trigger Rollout)
kubectl set image deployment/webapp app=nginx:1.28

# Scaling
kubectl scale deployment webapp --replicas=10

# Monitoring Rollout
kubectl rollout status deployment/webapp
kubectl rollout history deployment/webapp

# Rollback & Pause
kubectl rollout undo deployment/webapp
kubectl rollout pause deployment/webapp
kubectl rollout resume deployment/webapp
```
