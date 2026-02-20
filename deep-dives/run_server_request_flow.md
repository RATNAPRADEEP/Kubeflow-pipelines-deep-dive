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


## Internal Call Chain Breakdown

Within CreateRun():

1. Validate request payload
2. Extract namespace from context
3. Perform SubjectAccessReview (RBAC enforcement)
4. Call ResourceManager.CreateRun()
5. Persist run metadata via RunStore
6. Trigger workflow submission through workflow client
7. Return RunDetail response

---

## State Consistency Observation

Run metadata is persisted before workflow submission.

If workflow submission fails:
- Run record may exist in DB
- Workflow CR may not exist in Kubernetes
- UI can show inconsistent state until reconciliation

This indicates a potential improvement area:
Atomic persistence + workflow submission transaction handling.

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
