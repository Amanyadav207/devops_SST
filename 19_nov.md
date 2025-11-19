# Docker Networking – A Complete Guide

## 1. Network Fundamentals
*   **IP Address**: Unique identifier for a device on a network.
*   **NIC (Network Interface Card)**: Hardware connecting a computer to a network. Converts digital data to signals, handles MAC addresses.
*   **Ethernet Cable**: Physical connection (Cat5e, Cat6, etc.) carrying data.
*   **Network Switch (Layer 2)**: Connects devices within a LAN using MAC addresses. Efficiently forwards frames to specific ports.
*   **ARP (Address Resolution Protocol)**: Maps **IP Address → MAC Address**. Essential for local communication.
*   **Gateway**: The router exit point for traffic leaving the local subnet.
*   **NAT (Network Address Translation)**:
    *   **SNAT (Source NAT)**: Changes source IP (Private IP → Public IP) for internet access.
    *   **DNAT (Destination NAT)**: Changes destination IP (Public IP → Private IP) for port forwarding.
*   **Subnetting**: Dividing a network into smaller, secure segments (e.g., `/24` = 256 IPs).

## 2. Docker Networking Internals
Docker relies on Linux primitives:
*   **Network Namespaces**: Provides isolation (own IP, routing table, firewall) for each container.
*   **veth Pairs**: Virtual cables connecting a container (`eth0`) to the host bridge.
*   **Docker Bridge (`docker0`)**: Default virtual switch connecting containers on the host. Handles NAT for external access.

## 3. Docker Network Types
| Type | Description | Command |
| :--- | :--- | :--- |
| **Bridge** (Default) | Private internal network. Containers talk via IP or name (if custom). | `docker network create mynet` |
| **Host** | Shares host's network stack. No isolation, high performance. | `docker run --network host ...` |
| **None** | No networking. Fully isolated. | `docker run --network none ...` |
| **Overlay** | Multi-host networking for Swarm/Kubernetes. | `docker network create -d overlay ...` |
| **Macvlan** | Assigns unique MAC to container; appears as physical device. | `docker network create -d macvlan ...` |

## 4. Connectivity & Service Discovery
*   **Internet Access**: Containers use **SNAT** (via `iptables`) to masquerade as the host to access the internet.
*   **Port Mapping**: Exposes container ports to host using **DNAT**.
    *   `docker run -p 8080:80 nginx` (Host 8080 → Container 80).
*   **Service Discovery**:
    *   **Default Bridge**: Linking by IP (cumbersome).
    *   **Custom Bridge**: Built-in DNS resolution. Containers resolve each other by container name (e.g., `ping db`).

## 5. Essential Commands
```bash
# List all networks
docker network ls

# Create a custom bridge network
docker network create backend

# Run container in specific network
docker run -d --name db --network backend mysql

# Inspect network details (subnet, connected containers)
docker network inspect bridge
```
