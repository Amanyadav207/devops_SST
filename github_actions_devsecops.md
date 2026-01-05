# GitHub Actions DevSecOps Pipeline (Java + Docker)

## 1. Overview
This tutorial covers a **DevSecOps** pipeline for a Java Maven application.
**Goal**: Identify vulnerabilities *before* shipping code.
**Architecture**:
`Dev` → `GitHub Checks` (Build/Test/Scan) → `Docker Build` → `Container Scan` → `DockerHub`.

## 2. Prerequisites
1.  **Application**: Java Maven project with `Dockerfile`, listening on port 8080.
2.  **DockerHub**: Account + Access Token.
3.  **GitHub Secrets**:
    *   `DOCKERHUB_USERNAME`
    *   `DOCKERHUB_TOKEN`

## 3. Pipeline Configuration (`.github/workflows/ci.yml`)

### A. Triggers & Permissions
```yaml
on:
  push:
    branches: [ "master" ] # CI on push to master
  workflow_dispatch:       # Manual trigger

permissions:
  contents: read           # Read code
  security-events: write   # Upload security reports
```

### B. The 12 Stages (DevSecOps Flow)
| Stage | Action | Purpose |
| :--- | :--- | :--- |
| **1. Checkout** | `actions/checkout` | Pulls source code. |
| **2. Setup Java** | `actions/setup-java` | Installs Java & Maven with caching. |
| **3. Linting** | `mvn checkstyle:check` | Enforces code standards (Technical Debt). |
| **4. SAST (CodeQL)** | `github/codeql-action` | Scans source code for deep vulnerabilities (SQLi, etc.). |
| **5. SCA** | `Dependency-Check` | Checks libraries against CVE database (Supply Chain Security). |
| **6. Unit Test** | `mvn test` | Validates business logic. Fails builds on error. |
| **7. Build Artifact** | `mvn package` | Creates the JAR/WAR file. |
| **8. Docker Build** | `docker build` | Packages app into an immutable image. |
| **9. Image Scan** | `Trivy` | Scans container OS & layers for vulnerabilities. Fails on Critical/High. |
| **10. Smoke Test** | `docker run` + `curl` | Verifies container starts and responds. |
| **11. Login** | `docker/login-action` | Authenticates with DockerHub using Secrets. |
| **12. Keep/Push** | `docker push` | Uploads trusted, verified image to registry. |

### C. Key Concepts
*   **Shift-Left Security**: Security scans (SAST, SCA, Trivy) run *during* CI, not after deployment.
*   **Quality Gates**: Pipeline fails if tests fail OR critical vulnerabilities are found.
*   **Immutable Artifacts**: The Docker image tested is the exact same image pushed to prod.

## 4. Example Job Snippet
```yaml
jobs:
  ci-pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Security Scan (Container)
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myuser/myapp:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          exit-code: '1' # Fail on detection
          severity: 'CRITICAL,HIGH'

      # Upload to GitHub Security Tab
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```
