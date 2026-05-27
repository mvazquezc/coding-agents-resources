---
name: ocp-operator-troubleshoot
description: Troubleshoot OLM-managed operator deployments and runtime errors on OpenShift 4.12+. Supports live cluster (oc) and must-gather (omc) analysis with deep operator source code correlation.
---

Troubleshoot an OLM-managed operator: $ARGUMENTS

## Prerequisites

Verify the required CLI tools are installed:

1. Run `command -v oc > /dev/null 2>&1` and `command -v omc > /dev/null 2>&1`.
2. If **neither** is available, stop and tell the user: `Neither oc nor omc is installed. Install oc from https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/ and/or omc from https://github.com/gmeghnag/omc`
3. If only one tool is available, note which mode is possible (live-only or offline-only) and continue.

> **Note:** Throughout this skill, `CMD` refers to `oc` in live mode or `omc` in offline mode.

## Step 1: Mode Detection

Determine **live cluster** (`oc`) or **offline must-gather** (`omc`) mode:

1. If `$ARGUMENTS` contains a file system path, use **offline mode**. If it contains only an operator name/namespace, use **live mode**.
2. If ambiguous, ask the user: "Are you troubleshooting a live cluster or analyzing a must-gather bundle?"

**Live mode:** Run `oc whoami` to verify auth; capture `oc version` server version.
**Offline mode:** Run `omc use <path>`; capture OCP version via `omc get clusterversion version -o jsonpath='{.status.desired.version}'`.

## Step 2: Input Gathering

Determine the target operator and namespace from `$ARGUMENTS` or by guided discovery.

**If provided:** Validate by checking for a Subscription or CSV: `CMD get sub -n <namespace>` and `CMD get csv -n <namespace>`.

**Guided discovery:** Find operators in trouble:
```bash
CMD get csv -A -o json | jq -r '.items[] | select(.status.phase != "Succeeded") | "\(.metadata.namespace)\t\(.metadata.name)\t\(.status.phase)\t\(.status.reason // "N/A")"'
CMD get sub -A -o json | jq -r '.items[] | "\(.metadata.namespace)\t\(.metadata.name)\t\(.status.state // "Unknown")\t\(.status.currentCSV // "N/A")"'
```
Present results and ask the user to select the operator. Capture `OPERATOR_NAME`, `NAMESPACE`, `CSV_NAME`, `SUBSCRIPTION_NAME`.

## Step 3: Phase 1 -- OLM Object Triage

Check the OLM resource chain systematically:

### 3.1 CatalogSource
- Extract `spec.source` and `spec.sourceNamespace` from the Subscription.
- Check `.status.connectionState.lastObservedState` -- must be `READY`.
- Verify the catalog pod is running: `CMD get pods -n <sourceNamespace> -l olm.catalogSource=<source>`.

### 3.2 Subscription
- Check `.status.state` (expected: `AtLatestKnown`), `.status.currentCSV` vs `.status.installedCSV`, `.spec.installPlanApproval`, `.status.conditions`.
- If `currentCSV != installedCSV`: upgrade is stuck.

### 3.3 InstallPlan
- Find InstallPlans for the target CSV. Check `.spec.approved`, `.status.phase` (expected: `Complete`), `.status.conditions`.

### 3.4 OperatorGroup
- Verify exactly **one** OperatorGroup in the namespace. Multiple OperatorGroups cause OLM to reject installs.
- Check `.spec.targetNamespaces` and cross-reference with CSV install modes.

### 3.5 ClusterServiceVersion
- Check `.status.phase` (expected: `Succeeded`), `.status.reason`, `.status.message`.
- Check `.status.requirementStatus[]` for missing CRDs, ServiceAccounts, Deployments.
- Extract deployment names from `.spec.install.spec.deployments[].name` for Phase 2.
- Extract `repository` and `containerImage` annotations for Phase 3.

### OLM Summary
Summarize the chain: CatalogSource -> Subscription -> InstallPlan -> OperatorGroup -> CSV. Highlight broken links. If the issue is OLM-level, provide recommendations and ask if the user wants to continue to workload triage.

## Step 4: Phase 2 -- Workload Triage

### 4.1 Deployments
For each deployment from the CSV: check `.status.replicas` vs `.status.readyReplicas`, `.status.conditions` (Available, Progressing), resource requests/limits, scheduling constraints.

### 4.2 Pods
Check `.status.phase`, `.status.containerStatuses[].state` (CrashLoopBackOff, ImagePullBackOff, OOMKilled, etc.), `.restartCount`. For Pending pods, check events for `FailedScheduling`.

### 4.3 Container Logs
Pull logs with `CMD logs <pod> -n <NAMESPACE> -c <container> --tail=200` (and `--previous` for crash loops). Search for: `error`, `fatal`, `panic`, `connection refused`, `forbidden`, `Unauthorized`, `OOMKilled`, `Reconciler error`.

### 4.4 Events
Filter Warning events: `CMD get events -n <NAMESPACE> --sort-by='.lastTimestamp' -o json | jq -r '.items[] | select(.type != "Normal") | ...'`
Focus on: FailedScheduling, FailedCreate, Unhealthy, BackOff, FailedMount.

### 4.5 RBAC Verification
From the CSV permissions, extract the ServiceAccount and expected rules. Verify the SA exists, check Role/ClusterRole bindings. Cross-reference with any `forbidden` log errors.

### 4.6 Webhooks and CRDs
Check for webhooks owned by the CSV. Verify webhook Service endpoints exist and caBundle is populated. Verify CRDs are `Established`.

### Workload Summary
Group findings by severity (Critical / Warning / Info). If error logs were captured, proceed to Phase 3 for code correlation.

## Step 5: Phase 3 -- Deep Code Analysis

### 5.1 Source Repository
Auto-detect via CSV `repository` annotation or infer from `containerImage`. Ask the user to clone the repo and provide the local path.

### 5.2 Framework Detection
Identify operator type: Go operator-sdk/kubebuilder (`main.go`, `controllers/`), Ansible (`watches.yaml`, `roles/`), Helm (`helm-charts/`), Java, or Python.

### 5.3 Error Correlation (Go)
- Find reconciler files: search for `func.*Reconcile\(`.
- For each error message from logs, search the source for the exact string.
- Trace the call chain from `Reconcile` to the error origin.
- Check for: missing env vars (`os.Getenv`), expected Secrets/ConfigMaps, API version mismatches, finalizer issues, status update failures.

### 5.4 Error Correlation (Ansible/Helm)
- Ansible: check `roles/` task files for error messages, review `watches.yaml`.
- Helm: check `templates/` for rendering issues, review `values.yaml` defaults.

### Code Analysis Summary
For each traced error: report the error message, source file and line, trigger condition, and recommended fix.

## Step 6: Diagnostic Report

Produce a structured report:

```markdown
## Operator Diagnostic Report

| Field | Value |
|-------|-------|
| **Operator** | <OPERATOR_NAME> |
| **CSV** | <CSV_NAME> |
| **Namespace** | <NAMESPACE> |
| **Mode** | Live Cluster / Must-Gather |
| **OCP Version** | <version> |

### OLM Resource Chain Status

| Resource | Name | Status | Healthy |
|----------|------|--------|---------|
| CatalogSource | <name> | <state> | Yes/No |
| Subscription | <name> | <state> | Yes/No |
| InstallPlan | <name> | <phase> | Yes/No |
| OperatorGroup | <name> | <target> | Yes/No |
| CSV | <name> | <phase> | Yes/No |

### Findings

| # | Severity | Component | Finding | Recommendation |
|---|----------|-----------|---------|----------------|
| 1 | Critical/Warning/Info | <component> | <description> | <action> |

### Root Cause Analysis
<Narrative connecting findings across all phases.>

### Recommended Actions
<Numbered list of concrete actions, ordered by priority, with exact commands.>
```

## Command Equivalence

| Action | Live (`oc`) | Offline (`omc`) |
|--------|-------------|-----------------|
| Set context | `oc login <url>` | `omc use <path>` |
| Cluster version | `oc get clusterversion` | `omc get clusterversion` |
| Get resource | `oc get <resource> -n <ns>` | `omc get <resource> -n <ns>` |
| Pod logs | `oc logs <pod> -n <ns>` | `omc logs <pod> -n <ns>` |
| Previous logs | `oc logs <pod> --previous` | `omc logs <pod> --previous` |
| Events | `oc get events -n <ns>` | `omc get events -n <ns>` |
| JSON output | Add `-o json` | Add `-o json` |
