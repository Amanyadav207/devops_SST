# Kubernetes Services: Networking & Discovery

## 1. The Problem & Solution
*   **Problem**: Pods are ephemeral; their IPs change constantly (restart/scaling).
*   **Solution**: **Service**. Provides a stable Virtual IP (ClusterIP), DNS name, and load balancing across backing Pods.

## 2. How It Works
1.  **Selector**: Service matches Pods via labels (e.g., `app: backend`).
2.  **Kube-proxy**: Runs on every node. Creates iptables/IPVS rules.
3.  **Routing**: Traps traffic to the Service IP and forwards it to one of the backing Pods.

## 3. Service Types
| Type | Description | Use Case |
| :--- | :--- | :--- |
| **ClusterIP** (Default) | Internal IP only. Not reachable from outside. | Internal microservices, DBs. |
| **NodePort** | Opens a static port (`30000-32767`) on **every** Node's IP. | Local clusters, non-cloud external access. |
| **LoadBalancer** | Provisions a Cloud LB (AWS ELB, GCP) that points to NodePorts. | Production external access. |

## 4. Configuration Examples

### ClusterIP (Internal)
```yaml
apiVersion: v1
kind: Service
metadata: { name: backend-svc }
spec:
  type: ClusterIP
  selector: { app: backend }
  ports:
    - port: 80         # Service Port
      targetPort: 8080 # Container Port
```
*Access*: `http://backend-svc` (internally).

### NodePort (External - Manual)
```yaml
spec:
  type: NodePort
  selector: { app: backend }
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 32080 # Optional fixed port
```
*Access*: `http://<NodeIP>:32080`.

### LoadBalancer (External - Cloud)
```yaml
spec:
  type: LoadBalancer
  selector: { app: backend }
  ports:
    - port: 80
      targetPort: 8080
```
*Access*: `http://<EXTERNAL-IP>` (Provided by Cloud).

## 5. Advanced Concepts
*   **Headless Service** (`clusterIP: None`): No LB. DNS returns IPs of all Pods directly. Used for StatefulSets/DBs.
*   **DNS Naming**: `<service>.<namespace>.svc.cluster.local` (e.g., `redis.default.svc.cluster.local`).
*   **Multi-Port**: Expose HTTP (80) and HTTPS (443) in one Service.
*   **ExternalName**: Maps a local Service to an external DNS name (e.g., `db.aws.com`).
