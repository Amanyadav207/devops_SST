# Introduction to CI/CD

**Date:** December 17, 2024

## 1. What is CI/CD?
**CI/CD** (Continuous Integration & Continuous Delivery/Deployment) is a method to frequently deliver apps by automating development stages.

### Definitions
*   **Continuous Integration (CI)**: Automatically building and testing code changes.
    *   *Workflow*: Dev → Push → Repo → Build → Test → Report.
*   **Continuous Delivery (CD)**: Automatically preparing code for release (requires **manual approval** to deploy to Production).
*   **Continuous Deployment**: Automatically deploying to Production (**no manual approval**).

## 2. Pipeline Stages
1.  **Source**: Code commit triggers pipeline (Git).
2.  **Build**: Compile code, create artifacts, build Docker images.
3.  **Test**: Unit tests, Integration tests, Security scans.
4.  **Deploy**: Deploy to Staging/Production.
5.  **Monitor**: App monitoring, logs, alerts.

## 3. Best Practices
1.  **Automate Everything**: Build, Test, Deploy, Rollback.
2.  **Fast Feedback**: Builds < 10 mins.
3.  **Test Pyramid**: Many Unit Tests > Some Integration Tests > Few E2E Tests.
4.  **Version Everything**: Code, Config, Infrastructure (IaC).
5.  **Deployment Strategies**:
    *   **Blue-Green**: Two identical environments (switch traffic).
    *   **Canary**: Gradual rollout to small user subset.
    *   **Rolling**: Sequential updates replacement.

## 4. Popular Tools
| Type | Tools |
| :--- | :--- |
| **Cloud-Based** | GitHub Actions, GitLab CI/CD, CircleCI, Travis CI, AWS CodePipeline |
| **Self-Hosted** | Jenkins, TeamCity, Bamboo |

## 5. Benefits
*   **Faster Time to Market**: Frequent, automated releases.
*   **Improved Quality**: Early bug detection via automated testing.
*   **Reduced Risk**: Smaller changes are easier to rollback.
*   **Better Collaboration**: Shared responsibility and feedback.
