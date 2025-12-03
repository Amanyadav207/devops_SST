# Kubernetes ReplicaSet: Deep Dive

## 1. Overview
A **ReplicaSet (RS)** is a controller that ensures a specified number of Pod replicas are always running.
It continuously runs a **Reconciliation Loop**:
*   **Desired State**: `spec.replicas` (from YAML).
*   **Current State**: Number of running pods matching the label selector.
*   **Action**: Create or delete pods to match desired state.

## 2. The Workflow (Step-by-Step)
### Step 1: Submission
*   `kubectl apply -f rs.yaml` → **API Server**.
*   API Server authenticates, authorizes, validates, and stores RS in **etcd**.

### Step 2: Controller Action
*   **ReplicaSet Controller** (in Kube-Controller-Manager) watches API.
*   Sees new RS (Desired: 3, Current: 0).
*   Creates 3 Pod objects and sends them to API Server.
*   API Server stores Pods in **etcd** (Status: `Pending`, Node: `""`).

### Step 3: Scheduling
*   **Scheduler** sees `Pending` pods.
*   Assigns a node (e.g., `node1`).
*   Updates Pod spec in **etcd** (`spec.nodeName = node1`).

### Step 4: Execution
*   **Kubelet** on `node1` sees assigned Pods.
*   Pulls image, starts container (CRI).
*   Updates status to `Running` in API Server.

## 3. Failure Handling (Self-Healing)
The RS Controller always watches the loop.

| Scenario | Reaction |
| :--- | :--- |
| **Pod Crashes** | Kubelet reports `Failed`. RS sees 2/3 running. **Creates 1 new Pod**. |
| **Node Dies** | Node controller marks pods `Unknown` (>5m). RS **creates replacements** on healthy nodes. |
| **Manual Delete** | User deletes pod. RS sees 2/3. **Creates 1 new Pod**. |
| **Delete RS** | `kubectl delete rs`. RS Controller deletes all owner pods (cascade). |

## 4. Key Components Summary
*   **API Server**: Validates & persists state to etcd.
*   **etcd**: Stores desired (RS spec) and actual (Pod status) state.
*   **ReplicaSet Controller**: The "Brain" ensuring `Current == Desired`.
*   **Scheduler**: Assigns pods to nodes.
*   **Kubelet**: The "Muscle" running the containers.
