# Kubeflow Pipelines – Deep Technical Dive

This repository documents my structured study of Kubeflow Pipelines architecture, backend flow, state propagation, and debugging infrastructure.

The goal is to deeply understand the system before contributing improvements.

---

## 1. System Overview

Kubeflow Pipelines (KFP) enables defining and orchestrating ML workflows on Kubernetes using:

- Python SDK (pipeline authoring)
- Go API Server (control plane)
- Driver & Launcher (execution orchestration)
- Argo Workflows (workflow engine)
- MySQL (state persistence)
- ML Metadata (artifact tracking)
- React Frontend (monitoring & comparison UI)

---

## 2. End-to-End Execution Flow

1. User defines pipeline in Python
2. Compiler generates pipeline_spec (IR YAML)
3. API Server validates and stores run metadata
4. Driver converts spec → Argo Workflow CR
5. Kubernetes schedules Pods
6. Launcher handles input/output artifacts
7. Persistence Agent reconciles state
8. Frontend polls and renders execution graph

---

## 3. Backend Components Study

### API Server
- Handles CreateRun, GetRun
- Performs SubjectAccessReview for RBAC
- Persists run metadata

### Driver
- Patches Pod specs
- Applies resource configuration
- Handles workflow submission

### Launcher
- Downloads input artifacts
- Executes component function
- Uploads output artifacts

### Persistence Agent
- Polls Argo Workflow CR
- Updates run state in MySQL
- Handles eventual consistency

---

## 4. Observed Risk Areas

- Protobuf schema drift
- Swagger client generation mismatch
- Eventual consistency delays
- N+1 artifact fetch pattern in frontend
- Large MLMD artifact resolution loops

---

## 5. Debugging Challenges Identified

- Deep log inspection required
- Poor error summarization in UI
- Distributed failure tracing complexity
- Cross-layer error propagation ambiguity

---

## 6. Areas of Potential Contribution

- Structured error classification system
- Intelligent error summary panel
- Improved artifact batching
- Backend-to-frontend failure propagation improvements
- Observability enhancements

---

## 7. Next Study Areas

- Driver pod patch logic
- RunServer.go request lifecycle
- CompareV2.tsx artifact resolution
- MLMD event traversal path
