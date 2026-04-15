# OKE + Karpenter Namespace-Isolated Autoscaling - Agile Epics

## Epic 1: Foundation Infrastructure (OCI + OKE Always Free)
**Goal:** Stand up baseline OCI networking/IAM and an OKE cluster with one bootstrap node pool on Always Free A1 Flex.

### Stories
1. Create OCI network primitives (VCN, subnets, route tables, NSGs).
2. Provision OKE control plane and one managed bootstrap node pool (A1 Flex).
3. Configure IAM dynamic groups/policies for Karpenter operations.
4. Document tenancy/compartment prerequisites and quotas.

### Acceptance Criteria
- OKE cluster is reachable and healthy.
- Exactly one bootstrap node pool exists.
- Bootstrap nodes use A1 Flex Always Free-compatible config.
- IAM permissions for Karpenter are validated by OCI policy checks.

---

## Epic 2: Karpenter Installation and Baseline Configuration
**Goal:** Deploy Oracle Karpenter provider and establish reusable baseline node provisioning templates.

### Stories
1. Install Karpenter controller/provider in system namespace via Helm.
2. Create baseline `OCINodeClass` for A1 Flex workers.
3. Create baseline Karpenter `NodePool` for controlled test scheduling.
4. Validate scale-up and scale-down behavior with test workloads.

### Acceptance Criteria
- Karpenter pods are healthy.
- Test unschedulable pod triggers node provisioning.
- Provisioned nodes join cluster and become schedulable.
- Scale-down/disruption policy behaves as configured.

---

## Epic 3: Namespace-to-NodePool Automation
**Goal:** Automatically create dedicated Karpenter capacity definitions when a new namespace is created.

### Stories
1. Build namespace watcher controller.
2. Add opt-in namespace annotation contract:
   - `autoscale.oracle.com/enabled=true`
   - optional size and max-node annotations.
3. On namespace create, generate namespace-specific `NodePool` (+ `OCINodeClass` if needed).
4. On namespace delete, clean up generated Karpenter objects.
5. Ensure reconciliation is idempotent and safe on restart.

### Acceptance Criteria
- Opted-in namespace creation generates expected Karpenter resources.
- Non-opted namespaces do not create resources.
- Deletion removes namespace-scoped capacity objects.
- Repeated reconciles do not create duplicates or drift.

---

## Epic 4: Strict Namespace Isolation and 1 Pod Per Node Behavior
**Goal:** Ensure namespace workloads land only on their dedicated pool and scale node count with replicas.

### Stories
1. Apply namespace-specific taints on generated node pools.
2. Enforce workload tolerations + node affinity/selector contract.
3. Add required pod anti-affinity for app pods to achieve one workload pod per node.
4. Define default requests/limits guidance to avoid multi-pod packing.
5. Add policy checks/admission validation for required scheduling fields.

### Acceptance Criteria
- Workloads in namespace A cannot schedule on namespace B nodes.
- Missing toleration/selector causes pod to remain unschedulable (as expected).
- Replica increase leads to proportional node increase in namespace pool.
- Effective behavior is one application pod per node (excluding DaemonSets/system pods).

---

## Epic 5: Reliability, Governance, and Operational Readiness
**Goal:** Make the platform production-ready with guardrails, observability, and runbooks.

### Stories
1. Add namespace-level `ResourceQuota`/`LimitRange` defaults.
2. Create dashboards/alerts for pending pods, node churn, and provisioning failures.
3. Add runbooks for A1 capacity shortages and fallback procedures.
4. Add integration tests and failure-injection scenarios.
5. Define rollout plan and rollback criteria.

### Acceptance Criteria
- Alerting catches failed provisioning and prolonged pending pods.
- Guardrails prevent runaway namespace scaling.
- Runbook exists for OCI capacity constraints.
- End-to-end test suite validates happy path and key failure paths.

---

## Milestones
1. **M1:** Epics 1-2 complete (cluster + Karpenter baseline).
2. **M2:** Epic 3 complete (namespace automation live in non-prod).
3. **M3:** Epic 4 complete (hard isolation + one workload pod per node).
4. **M4:** Epic 5 complete (operational readiness and sign-off).

---

## Assumptions
- A1 Flex capacity is available in target region.
- Workloads are ARM64-compatible.
- “1 pod per node” applies to application pods, not DaemonSet/system pods.
- Namespace onboarding is opt-in via annotation.
