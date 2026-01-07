# Continuous Deployment (CD) Implementation

**Date:** January 7, 2025

## 1. Overview
**Continuous Deployment (CD)** automates the release of applications to production after successful testing.
**Flow:** `CI (Build)` → `Artifact` → `Staging` → `Tests` → `Production`.

## 2. Deployment Agents (Runners)
**Self-Hosted Runners** allow running workflows on your own infrastructure (e.g., private cloud, on-prem).
*   **Setup**: Download runner binary → Configure with Token → Run.

## 3. Deployment Methods

### A. Kubernetes (kubectl)
Directly update image in Deployment.
```yaml
jobs:
  deploy:
    steps:
    - run: kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}
```

### B. Helm
Manage releases using Charts.
```yaml
- run: helm upgrade --install myapp ./chart --set image.tag=${{ github.sha }}
```

## 4. Deployment Strategies
| Strategy | Description | Config |
| :--- | :--- | :--- |
| **Rolling** | Gradual update (zero downtime). | `strategy: RollingUpdate` |
| **Blue-Green** | Switch traffic between 2 envs (Instant). | Patch Service selector (Blue -> Green). |
| **Canary** | Rollout to small % of users first. | separate Deployment with small replica count. |

## 5. Environment Management
Use GitHub jobs with `if` conditions to promote artifacts across environments.
```yaml
jobs:
  staging:
    if: github.ref == 'refs/heads/develop'
  production:
    if: github.ref == 'refs/heads/main'
    needs: staging
```

## 6. Secrets & Safety
*   **Secrets**: Store Kubeconfig in GitHub Secrets. Decode in workflow.
*   **Health Checks**: `kubectl rollout status` to wait for success.
*   **Rollback**: `kubectl rollout undo` if deployment fails.
