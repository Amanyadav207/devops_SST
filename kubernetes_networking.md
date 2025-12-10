# Kubernetes Networking Model: Deep Dive

## 1. Golden Rules
Kubernetes imposes 3 fundamental networking rules:
1.  **Pod-to-Pod**: All Pods can communicate with all other Pods without NAT.
2.  **Node-to-Pod**: All Nodes can communicate with all Pods without NAT.
3.  **IP Consistency**: A Pod sees itself with the same IP that others see it with.

## 2. Communication Layers

### A. Container-to-Container (In Pod)
*   **Mechanism**: Shared Network Namespace.
*   **Method**: `localhost`.
*   **Shared**: IP, Ports, Loopback.

### B. Pod-to-Pod (Same Node)
*   **Mechanism**: veth pairs + Linux Bridge (`cbr0`).
*   **Flow**: Pod A `eth0` → `veth` → Bridge → ARP → `veth` → Pod B `eth0`.

### C. Pod-to-Pod (Cross Node)
*   **Mechanism**: Routing via CNI (Container Network Interface).
*   **Key**: Each Node has a distinct Pod CIDR (e.g., Node1: `10.0.1.0/24`, Node2: `10.0.2.0/24`).
*   **Flow**: Pod A → Bridge → Node A `eth0` → Network Router → Node B `eth0` → Bridge → Pod B.
*   **CNIs**:
    *   **AWS VPC CNI**: Real VPC IPs (No overlay, high performance).
    *   **Calico/Flannel/Weave**: Uses overlays (encapsulation) or BGP.

### D. Pod-to-Service (ClusterIP)
*   **Problem**: Pod IPs change. Service provides a stable VIP.
*   **Implementation**: **kube-proxy** (iptables or IPVS).
    *   **iptables**: Randomly DNATs packets to backend Pods.
    *   **IPVS**: Kernel-level LB, scalable, faster.
*   **DNS**: `CoreDNS` resolves `mysvc.ns.svc.cluster.local` to ClusterIP.

### E. Egress (Pod → Internet)
*   **Problem**: Internet GW doesn't know Pod IPs.
*   **Solution**: **SNAT** (Source NAT).
*   **Flow**: Packet leaves Node → Source IP rewritten from PodIP to **NodeIP**.

### F. Ingress (Internet → Pod)
1.  **LoadBalancer (L4)**: Cloud LB (AWS/GCP) → NodePort → Pod. Simple TCP/UDP.
2.  **Ingress Controller (L7)**: Host/Path routing (e.g., NGINX, Traefik, ALB). HTTP/HTTPS termination.

## 3. Summary Cheat Sheet
| Layer | Mechanism |
| :--- | :--- |
| **Container-to-Container** | `localhost` (Shared Namespace) |
| **Pod-to-Pod** | Routing via CNI (veth + bridge + routes) |
| **Pod-to-Service** | `iptables`/`IPVS` (DNAT) |
| **Egress** | SNAT (Masquerade as Node IP) |
| **Ingress** | LoadBalancer or Ingress Controller |
