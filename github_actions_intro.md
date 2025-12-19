# Introduction to GitHub Actions

## 1. What is GitHub Actions?
**GitHub Actions** automates tasks within your repository (CI/CD).
*   **Trigger**: Event based (e.g., Code push, PR creation).
*   **Action**: Run workflows defined in YAML (e.g., Build app, Run tests).

## 2. Terminology
| Term | Meaning |
| :--- | :--- |
| **Workflow** | The automation process (YAML file). |
| **Job** | A set of steps executing on a runner. |
| **Step** | A single command or action. |
| **Runner** | The server (VM) that executes the job (Linux, Mac, Windows). |
| **Action** | Reusable plugin/code (e.g., `actions/setup-java`). |

## 3. Workflow Example: Java Build
**Goal**: Build a Java/Maven project whenever code is pushed to `main`.

### A. Folder Structure
```text
java-github-actions-demo/
├── src/Main.java
├── pom.xml
└── .github/workflows/build.yml  <-- Workflow file
```

### B. The Workflow (`build.yml`)
```yaml
name: Java Build Workflow

on:
  push:
    branches: [ "main" ]  # Trigger on push to main

jobs:
  build:                  # Job Name
    runs-on: ubuntu-latest # Runner Type

    steps:
    - name: Checkout code
      uses: actions/checkout@v4   # Pre-built action to clone repo

    - name: Set up JDK 17
      uses: actions/setup-java@v4 # Pre-built action to install Java
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Build with Maven
      run: mvn clean package      # Run shell command
```

## 4. Execution Flow
1.  **Event**: Push to `main` detected.
2.  **Runner**: GitHub provisions a `ubuntu-latest` VM.
3.  **Checkout**: Repo code downloaded to VM.
4.  **Setup**: Java 17 installed.
5.  **Build**: `mvn clean package` runs.
6.  **Result**: Green check ✅ (Success) or Red cross ❌ (Fail) in "Actions" tab.

## 5. Common Common Mistakes
*   **Missing `.github/workflows` folder**: Directory structure must be key-exact.
*   **YAML Indentation**: Use **2 spaces**, never tabs.
*   **Wrong Java Version**: `pom.xml` version must match `setup-java` version.
