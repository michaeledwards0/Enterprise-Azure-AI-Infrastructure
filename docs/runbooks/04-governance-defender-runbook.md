<div align="center">

# Phase 4 Runbook: Governance & Defender for Cloud
### Step-by-Step Implementation Guide for Continuous Governance, Posture Management, and Logging Readiness

</div>

---

## Purpose

This runbook documents the implementation steps used to extend the Contoso AI Labs governance baseline, enable Microsoft Defender for Cloud workload protection, establish a Secure Score baseline, configure centralized diagnostic logging, and prepare the environment for Phase 5 detection engineering.

Use the accompanying [Phase 4 Portfolio Case Study](../04-governance-defender.md) for the executive summary, architecture, engineering decisions, results, and lessons learned.

---

## Scope

This runbook covers:

- Azure Policy compliance review
- Microsoft Defender for Cloud configuration
- Defender for AI Services
- Defender for Key Vault
- Defender for Servers
- Secure Score baseline capture
- Log Analytics Workspace deployment
- Azure AI Services diagnostic settings
- Key Vault diagnostic settings
- Azure Activity Log export
- Microsoft Entra ID diagnostic settings
- Storage Account archival
- Governance Reader role assignment
- Compliance validation
- Troubleshooting and completion checks

---

## Prerequisites

- Phases 1 through 3 complete
- `rg-secure-ai-prod` exists
- `ai-contoso-openai` exists
- `kv-contoso-ai` exists
- `vm-ai-workload-test` exists if Defender for Servers is being enabled
- Storage Account available for diagnostic archival
- Permissions to manage Azure Policy, Defender for Cloud, diagnostic settings, RBAC, and Log Analytics
- Microsoft Entra permissions sufficient to configure identity diagnostic settings

---

## Resource Naming

| Resource | Name |
|---|---|
| Production resource group | `rg-secure-ai-prod` |
| Azure AI Services resource | `ai-contoso-openai` |
| Key Vault | `kv-contoso-ai` |
| Test VM | `vm-ai-workload-test` |
| Log Analytics Workspace | `law-contoso-ai` |
| Diagnostic archive Storage Account | Existing project Storage Account |
| Governance role | `Reader` |

---

# 1. Review Existing Azure Policy Assignments

1. Open **Azure Portal → Policy**.
2. Select **Assignments**.
3. Review the assignments created in earlier phases.
4. Confirm the intended scope for each assignment:
   - Subscription
   - Resource group
   - Specific excluded or exempted scopes
5. Verify that `rg-secure-ai-prod` is included in scope.
6. Document any assignments that were temporarily removed, exempted, or changed during development.

Typical controls to review include:

- Public network access restrictions
- Managed identity requirements
- Public IP restrictions
- Diagnostic-settings recommendations
- Azure AI Services security recommendations

## Validation

Confirm:

- The expected assignments are visible
- The scope includes the intended resources
- Any exemptions are narrowly scoped
- No broad subscription exemption was created unnecessarily

---

# 2. Review Azure Policy Compliance

1. Open **Policy → Compliance**.
2. Filter by:
   - Subscription
   - Resource group: `rg-secure-ai-prod`
3. Review:
   - Compliant resources
   - Non-compliant resources
   - Exempt resources
   - Unknown or not-started states
4. Open each relevant policy to inspect the exact failing condition.
5. Distinguish between:
   - A true configuration issue
   - A delayed compliance evaluation
   - A recommendation deferred by design
   - A policy conflict with Azure AI Foundry or private networking

## Force or Trigger Reevaluation

If a remediated resource remains non-compliant:

1. Confirm the effective resource configuration.
2. Wait for policy evaluation to refresh.
3. Use the policy compliance view to trigger a scan where available.
4. Recheck the resource after evaluation completes.

## Validation

Confirm:

- The compliance state is documented
- Each non-compliant item has a remediation decision
- Deferred findings include a technical reason
- Policy state matches the effective resource configuration

---

# 3. Enable Microsoft Defender for Cloud

1. Open **Microsoft Defender for Cloud**.
2. Go to **Environment settings**.
3. Select the project subscription.
4. Open **Defender plans**.
5. Enable the plans required for this environment.

## 3.1 Defender for AI Services

Enable:

- **Defender for AI Services**

Use this plan to add workload protection and recommendations for the Azure AI resource.

## 3.2 Defender for Key Vault

Enable:

- **Defender for Key Vault**

Use this plan to monitor suspicious access and security events involving Key Vault.

## 3.3 Defender for Servers

Enable:

- **Defender for Servers**
- Use the lowest practical plan that satisfies the lab objective and budget

For a small portfolio environment, Plan 1 may be sufficient unless a later detection scenario requires additional features.

6. Save the Defender plan configuration.
7. Wait for the subscription configuration to update.

## Validation

Confirm:

- Defender for AI Services shows Enabled
- Defender for Key Vault shows Enabled
- Defender for Servers shows Enabled
- The correct subscription is selected
- Estimated costs are understood and documented

---

# 4. Capture the Secure Score Baseline

1. Open **Microsoft Defender for Cloud → Secure Score**.
2. Record:
   - Current Secure Score
   - Maximum available score
   - Percentage score
   - Top security recommendations
3. Review each recommendation for:
   - Security impact
   - Cost
   - Operational dependency
   - Risk of workload disruption
4. Identify quick wins that can be remediated safely during this phase.
5. Document recommendations deferred to later phases.

Examples of findings that may be deferred:

- Disable local authentication after Entra-based authentication is validated
- Additional network restrictions that interfere with Foundry access
- Backup-related controls scheduled for Phase 6
- Sentinel or analytics-rule controls scheduled for Phase 5

## Validation

Confirm:

- A baseline score is recorded
- High-impact recommendations are reviewed
- Deferred items include a rationale
- No recommendation is remediated blindly

---

# 5. Create the Log Analytics Workspace

1. Open **Azure Portal → Log Analytics workspaces → Create**.
2. Configure:
   - **Subscription:** Project subscription
   - **Resource group:** `rg-secure-ai-prod`
   - **Name:** `law-contoso-ai`
   - **Region:** Same project region where practical
3. Review and create.
4. Open the workspace after deployment.
5. Review:
   - Workspace ID
   - Region
   - Pricing tier
   - Retention settings
6. Set an appropriate retention period for the project:
   - Example: 30 days

## Validation

Confirm:

- `law-contoso-ai` is deployed
- The workspace is in the intended region
- Retention is configured
- The workspace is accessible for later KQL queries

---

# 6. Configure Azure AI Services Diagnostic Settings

1. Open `ai-contoso-openai`.
2. Go to **Monitoring → Diagnostic settings**.
3. Select **Add diagnostic setting**.
4. Name the setting:
   - Example: `diag-ai-contoso-openai`
5. Enable all available relevant categories.

Recommended selections:

- **Audit**
- **AllLogs**
- **AllMetrics**

Depending on the service, this may include:

- Audit logs
- Request and response logs
- Azure OpenAI request usage
- Trace logs
- Platform metrics

6. Select **Send to Log Analytics workspace**.
7. Choose:
   - Subscription
   - `law-contoso-ai`
8. Select **Archive to a storage account**.
9. Choose the existing project Storage Account.
10. Save.

## Validation

Confirm:

- Diagnostic setting is saved
- Log Analytics destination is correct
- Storage destination is correct
- Audit, logs, and metrics are enabled
- The setting applies to `ai-contoso-openai`

---

# 7. Configure Key Vault Diagnostic Settings

1. Open `kv-contoso-ai`.
2. Go to **Monitoring → Diagnostic settings**.
3. Select **Add diagnostic setting**.
4. Name the setting:
   - Example: `diag-kv-contoso-ai`
5. Enable the available audit and metrics categories.
6. Select **Send to Log Analytics workspace**.
7. Choose `law-contoso-ai`.
8. Select **Archive to a storage account**.
9. Choose the project Storage Account.
10. Save.

## Validation

Confirm:

- Key Vault audit logs are enabled
- Metrics are enabled where available
- Log Analytics and Storage destinations are correct

---

# 8. Export Azure Activity Logs

Azure Activity Logs capture subscription-level control-plane events such as resource creation, deletion, role assignment changes, policy operations, and service-health events.

1. Open **Subscriptions**.
2. Select the project subscription.
3. Go to **Activity log**.
4. Select **Export Activity Logs** or **Diagnostic settings**.
5. Create a diagnostic setting:
   - Example: `diag-subscription-activity`
6. Enable relevant categories:

- Administrative
- Security
- Service Health
- Resource Health
- Alert
- Recommendation
- Policy
- Autoscale

7. Select **Send to Log Analytics workspace**.
8. Choose `law-contoso-ai`.
9. Optionally archive to the Storage Account.
10. Save.

## Validation

Confirm:

- The setting exists at subscription scope
- Administrative and Security categories are enabled
- The Log Analytics destination is correct

---

# 9. Configure Microsoft Entra ID Diagnostic Settings

Microsoft Entra diagnostic settings are configured separately from Azure resource diagnostic settings.

1. Open **Microsoft Entra ID**.
2. Go to **Monitoring & health → Diagnostic settings**.
3. Select **Add diagnostic setting**.
4. Name the setting:
   - Example: `diag-entra-contoso-ai`
5. Enable the categories available in the tenant.

Recommended categories include:

- `AuditLogs`
- `SignInLogs`
- `NonInteractiveUserSignInLogs`
- `ServicePrincipalSignInLogs`
- `ManagedIdentitySignInLogs`
- `ProvisioningLogs`
- `RiskyUsers` if available
- `UserRiskEvents` if available

6. Select **Send to Log Analytics workspace**.
7. Choose `law-contoso-ai`.
8. Save.

## Validation

Confirm:

- Entra diagnostic setting is saved
- Identity logs target `law-contoso-ai`
- The selected categories are supported by the tenant and license

---

# 10. Configure Supporting Resource Diagnostics

Where available and relevant, configure diagnostic settings for supporting resources such as:

- Network Security Groups
- Azure Bastion
- Virtual Network gateways
- Storage Account
- Private DNS or other monitored services
- Defender-related data sources

For each resource:

1. Open **Monitoring → Diagnostic settings**.
2. Enable relevant logs and metrics.
3. Send operational data to `law-contoso-ai`.
4. Archive where long-term retention is useful.
5. Save.

Do not enable every category blindly. Select logs that support the project's detection and investigation goals.

---

# 11. Configure NSG Flow Logs and Traffic Analytics

If supported in the subscription and region:

1. Open **Network Watcher**.
2. Go to **Flow logs**.
3. Select the NSG protecting the AI workload subnet.
4. Enable flow logs.
5. Configure:
   - Storage Account destination
   - Appropriate retention
6. Enable **Traffic Analytics**.
7. Select:
   - `law-contoso-ai`
8. Choose an appropriate processing interval.
9. Save.

## Validation

Confirm:

- Flow logs are enabled for the intended NSG
- Raw flow data targets the Storage Account
- Traffic Analytics targets `law-contoso-ai`

> NSG Flow Logs record network-flow data. Azure Activity Logs record control-plane changes to the NSG. Both provide different investigative value.

---

# 12. Assign Governance Reader Role

1. Open `rg-secure-ai-prod`.
2. Go to **Access control (IAM)**.
3. Select **Add → Add role assignment**.
4. Choose:
   - Role: `Reader`
5. Assign the role to:
   - A dedicated governance identity, audit group, or the project test identity
6. Scope the assignment to:
   - `rg-secure-ai-prod`
7. Save.
8. Wait for propagation.

## Validation

Confirm the assigned identity can:

- View resources
- Review Azure Policy compliance
- Review Secure Score
- Review Defender recommendations

Confirm the identity cannot:

- Modify resources
- Change role assignments
- Change Defender plans
- Change policy assignments

---

# 13. Validate Log Ingestion

Allow time for diagnostic data to arrive.

1. Open `law-contoso-ai`.
2. Go to **Logs**.
3. Run simple discovery queries.

Example:

```kusto
search *
| summarize Count = count() by $table
| order by Count desc
```

Review whether expected tables are appearing.

Potential tables include:

- `AzureActivity`
- `AuditLogs`
- `SigninLogs`
- `AzureDiagnostics`
- Resource-specific tables
- Defender or security-related tables

## Validate Azure Activity

```kusto
AzureActivity
| take 20
```

## Validate Entra Audit Logs

```kusto
AuditLogs
| take 20
```

## Validate Sign-In Logs

```kusto
SigninLogs
| take 20
```

If a table does not exist yet:

- Generate a relevant event
- Wait for ingestion
- Verify the diagnostic setting
- Confirm the correct workspace was selected

---

# 14. Document Deferred Security Recommendations

Create a small remediation register for findings that should not be completed during this phase.

Recommended fields:

| Recommendation | Current State | Reason Deferred | Planned Phase |
|---|---|---|---|
| Disable local authentication | API-key connection still active | Entra migration must be validated first | Phase 4 enhancement or later hardening |
| Enforce stricter public-network policy | Foundry access conflict | Final access path must be validated | Later governance enforcement |
| Enable backup controls | Backup not yet configured | Scheduled work | Phase 6 |
| Enable Sentinel analytics rules | Telemetry foundation only | Detection engineering not yet built | Phase 5 |

This prevents valid recommendations from being ignored while preserving workload stability.

---

# 15. Completion Checklist

## Azure Policy

- [ ] Existing assignments reviewed
- [ ] Scope verified
- [ ] Compliance dashboard reviewed
- [ ] Non-compliant resources investigated
- [ ] Exemptions documented
- [ ] Deferred controls recorded

## Defender for Cloud

- [ ] Defender for AI Services enabled
- [ ] Defender for Key Vault enabled
- [ ] Defender for Servers enabled
- [ ] Secure Score baseline captured
- [ ] Recommendations reviewed
- [ ] Cost impact documented

## Log Analytics

- [ ] `law-contoso-ai` deployed
- [ ] Retention configured
- [ ] Workspace accessible
- [ ] Test queries executed

## Diagnostic Settings

- [ ] Azure AI Services diagnostics enabled
- [ ] Audit enabled
- [ ] AllLogs enabled
- [ ] AllMetrics enabled
- [ ] Key Vault diagnostics enabled
- [ ] Azure Activity Logs exported
- [ ] Microsoft Entra logs exported
- [ ] Storage archival configured
- [ ] Supporting-resource diagnostics reviewed

## Governance Access

- [ ] Reader role assigned
- [ ] Scope limited to `rg-secure-ai-prod`
- [ ] Read-only access validated
- [ ] No modification rights granted

## Detection Readiness

- [ ] Azure Activity data visible
- [ ] Entra audit data visible
- [ ] Sign-in data visible where licensed
- [ ] Resource logs visible
- [ ] Environment ready for Microsoft Sentinel onboarding

## Evidence

- [ ] Policy compliance screenshot captured
- [ ] Defender plans screenshot captured
- [ ] Secure Score screenshot captured
- [ ] Reader role screenshot captured
- [ ] Governance baseline screenshot captured
- [ ] Log Analytics Workspace screenshot captured
- [ ] Azure AI diagnostic settings screenshot captured
- [ ] Dual-destination screenshot captured
- [ ] Activity Log export screenshot captured
- [ ] Entra diagnostic settings screenshot captured

---

# 16. Troubleshooting

## Policy remains non-compliant after remediation

Check:

- The actual resource setting
- Policy definition details
- Assignment scope
- Exemptions
- Evaluation delay
- Whether the policy requires a specific diagnostic category or destination

Do not repeatedly change a correct resource solely because the portal has not refreshed.

## Defender plan does not appear enabled

Check:

- Correct subscription selected
- Defender plan saved successfully
- Required permissions
- Subscription eligibility
- Portal refresh and propagation time

## Diagnostic setting cannot be saved

Check:

- Contributor or Monitoring Contributor permissions
- Workspace and resource regions
- Storage Account availability
- Supported log categories
- Whether a setting with the same name already exists

## No logs appear in Log Analytics

Check:

- Correct workspace selected
- Diagnostic setting is enabled
- Relevant events have occurred
- Ingestion delay
- Table naming
- Workspace time range
- Tenant licensing for Entra log categories

## Entra diagnostic categories are unavailable

Check:

- Tenant license level
- Required Entra role
- Whether the category is supported in the tenant
- Whether another diagnostic setting already exports the logs

## Reader cannot view security posture

Check:

- Role-assignment scope
- RBAC propagation
- Whether an additional security-reader role is required for a specific Defender view
- Whether the identity is using the correct tenant and subscription

## Foundry access breaks after policy enforcement

Check:

- Public-network restriction policy
- Policy assignment mode
- Scope and exemption
- Private endpoint and DNS path
- Whether Foundry still depends on a public or API-key connection

Use Audit mode during development where enforcement would interrupt the active workload.

---

## Related Documentation

- [Phase 4 Portfolio Case Study](../04-governance-defender.md)
- [Phase 3 — Azure AI Services Deployment](../03-openai-deployment.md)
- [Phase 5 — Detection Engineering](../05-detection-engineering.md)
- [Project Overview](../../README.md)

---

<div align="center">

**Runbook complete — verify policy state, Defender coverage, RBAC, and telemetry ingestion before building Sentinel detections.**

</div>
