# RunServer Request Lifecycle – Deep Dive

## Entry Point

The RunServer handles pipeline run creation requests through:

CreateRun()

Located in:
backend/src/apiserver/server/run_server.go

---

## High-Level Flow

1. gRPC request received by API Server
2. RunServer validates input
3. ResourceManager performs:
   - Namespace validation
   - Experiment lookup
4. Run metadata persisted in MySQL
5. DriverManager triggered
6. Workflow CR submitted to Kubernetes
7. Persistence Agent begins reconciliation loop
8. Frontend later polls run status

---

## Key Observations

- Run metadata persistence occurs before workflow submission
- Failure between DB write and workflow submission can create orphaned runs
- Error propagation back to UI depends on correct status update in persistence agent
- RBAC enforcement happens early via SubjectAccessReview

---

## Potential Failure Points

- Invalid pipeline spec validation
- Experiment ID mismatch
- Workflow submission failure
- Kubernetes API timeout
- Persistence agent lag causing stale UI state
