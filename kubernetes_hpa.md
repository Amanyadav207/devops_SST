# Kubernetes Horizontal Pod Autoscaler (HPA)

## 1. Overview
**HPA** automatically scales the number of Pod replicas based on observed CPU utilization (or other metrics).
*   **Trigger**: User defined threshold (e.g., CPU > 50%).
*   **Mechanism**: Increases/Decreases replicas in Deployment/ReplicaSet.

## 2. Prerequisites (Critical)
1.  **Metrics Server**: Must be installed. HPA relies on it for resource stats.
2.  **Resource Requests**: Pod containers **MUST** have `resources.requests.cpu` defined. HPA calculates usage as `CurrentUsage / Request`.

## 3. Workflow
`User Load` → `Service` → `Pods` → `Metrics Server` → `HPA Controller` → `Scale Up/Down`

## 4. Implementation Steps

### Step 1: Install Metrics Server
```bash
kubectl apply -f https://github.com/vilasvarghese/docker-k8s/blob/master/yaml/metricServer/metric-server.yaml
# Verify
kubectl top nodes
```

### Step 2: Deployment (With CPU Requests)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: hpa-demo }
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: demo
        image: k8s.gcr.io/hpa-example
        resources:
          requests: { cpu: 200m } # Mandatory for HPA
          limits: { cpu: 500m }
```

### Step 3: Define HPA
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata: { name: hpa-demo }
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## 5. Testing & Monitoring
### Generate Load
```bash
# Run a loop to hit the service
kubectl run -i --tty load-gen --image=busybox -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-demo; done"
```

### Monitor Scaling
```bash
kubectl get hpa -w
kubectl get deploy hpa-demo -w
```

## 6. Troubleshooting
| Issue | Cause | Fix |
| :--- | :--- | :--- |
| **Targets: `<unknown>/50%`** | Metrics Server missing or delayed. | Install Metrics Server / Wait. |
| **No Scaling** | CPU Requests missing in Deployment. | Add `resources.requests.cpu`. |
