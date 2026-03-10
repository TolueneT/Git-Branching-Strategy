# **Table of Content**

1. Introduction
2. System Overview
3. Repository Structure
4. Branching Strategy
5. CI/CD Pipeline Design
6. Environment Strategy
7. Release Workflow
8. Hotfix Workflow
9. Branch Protection Rules
10. Architecture Diagram

# **Introduction**

This document defines the Git branching strategy and CI/CD workflow for the TESXpress microservices e-commerce platform. The objective is to enable safe collaboration among developers, enforce automated testing, and support reliable deployments across development, staging, and production environments.

The system will use:

1. Git for version control
2. GitHub Actions for CI/CD
3. Docker for containerization
4. Kubernetes for deployment orchestration

# **System Overview**

The application is an e-commerce platform composed of multiple microservices.
Each service is developed independently but deployed through a centralized CI/CD pipeline.
The list of services includes:

1. user-service
2. product-service
3. order-service
4. payment-service
5. notification-service

| Service              | Responsibility                   |
| -------------------- | -------------------------------- |
| user-service         | authentication and user profiles |
| product-service      | product catalog                  |
| order-service        | order management                 |
| payment-service      | payment processing               |
| notification-service | email and alerts                 |

# **Define Repository Structure**

The repository follows a monorepo architecture, where all microservices are stored in a single repository. This simplifies dependency management, enables centralized CI/CD pipelines, and ensures consistent deployment workflows across services.

```
TESXpress-platform/
│
├── user-service
├── product-service
├── order-service
├── payment-service
├── notification-service
│
└── infrastructure
     └── kubernetes-manifests
```

The infrastructure directory contains Kubernetes manifests and deployment configurations used by the CI/CD pipeline.

# **Design the Branching Strategy**

GitFlow was chosen because it provides a structured workflow for teams working on multiple features simultaneously. It separates ongoing development from production-ready code and provides dedicated branches for releases and hotfixes.

**Main Branch**

```
main
```

Purpose:

- Production-ready code.

Rules:

- protected
- no direct commits
- pull request required

**Develop Branch**

```
develop
```

Purpose:

- Integration branch where completed features are merged.
- Used for development environment deployment.

**Feature Branches**

```
feature/*
```

Purpose:

- Used by developers to implement new features.

**Release Branches**

```
release/*
```

Example:

```
release/v1.0
```

Purpose:

- Prepare code for production.
- Deployed to staging environment.

**Hotfix Branches**

```
hotfix/*
```

Example:

```
hotfix/payment-bug
```

Purpose:

- Urgent production fixes.

**The CI/CD Workflow Design**
The pipeline is triggered automatically based on the branch being pushed to.

| Branch      | Pipeline Action      |
| ----------- | -------------------- |
| `feature/*` | Run tests            |
| `develop`   | Deploy to dev        |
| `release/*` | Deploy to staging    |
| `main`      | Deploy to production |

Each pipeline run executes the following stages in order:

1. **Code Checkout** – pulls the latest code from the repository
2. **Install Dependencies** – installs all required packages
3. **Run Unit Tests** – validates code correctness before building
4. **Build Docker Image** – packages the application into a container
5. **Push Image to Registry** – uploads the image to the container registry
6. **Deploy to Kubernetes** – applies the updated manifests to the cluster

### Tools Used

- **Docker** – builds and packages container images
- **Kubernetes** – orchestrates and manages container deployments

**Defining the Environments**
Code is deployed to different environments depending on the stage of development:

| Environment    | Purpose                   |
| -------------- | ------------------------- |
| **Dev**        | Testing new features      |
| **Staging**    | Pre-production validation |
| **Production** | Live system               |

### Deployment Flow

Code progresses through the following branches before reaching production:

```
feature → develop → release → main
```

**The Release Workflow**
Releases follow a structured workflow to ensure code is tested
at every stage before reaching production:

1. **Developer completes feature** – feature branch is ready for review
2. **Pull request to `develop`** – code is submitted for review
3. **CI tests run** – automated tests validate the changes
4. **Merge to `develop`** – approved code is merged into the develop branch
5. **Deploy to development** – changes are deployed to the dev environment
6. **Create release branch** – a `release/*` branch is cut from develop
7. **Deploy to staging** – release branch is deployed for pre-production testing
8. **QA testing** – quality assurance validates the release candidate
9. **Merge to `main`** – approved release is merged into the main branch
10. **Deploy to production** – final deployment to the live system

**How production bugs are fixed**
When a bug is discovered in production, it follows an expedited
workflow that bypasses the normal release cycle:

1. **Production bug discovered** – an issue is identified in the live system
2. **Create hotfix branch from `main`** – a `hotfix/*` branch is cut directly from main
3. **Apply fix** – the bug is resolved in the hotfix branch
4. **Run CI tests** – automated tests validate the fix
5. **Merge into `main`** – approved fix is merged back into main
6. **Deploy to production** – fix is immediately deployed to the live system
7. **Merge back to `develop`** – fix is back-ported to keep develop up to date

### Hotfix Flow

```
main
 │
 ├──> hotfix/* ──> Apply Fix ──> CI Tests
 │                                   │
 │◄──────────── Merge to main ───────┘
 │
 ├──> Deploy to Production
 │
 └──> Merge back to develop
```

**Architecture Diagram**
The following flow represents the full lifecycle of code from
development to production:

```
Developer
    │
    ▼
Feature Branch
    │
    ▼
Pull Request
    │
    ▼
CI Pipeline
    │
    ▼
Merge to Develop
    │
    ▼
Deploy to Dev
    │
    ▼
Release Branch
    │
    ▼
Deploy to Staging
    │
    ▼
Merge to Main
    │
    ▼
Deploy to Production
```

**Conclusion**

This branching strategy ensures safe collaboration among developers while enabling automated CI/CD pipelines. It protects production environments while allowing rapid feature development and controlled releases.
