<div align="center">

# Phase 9 Runbook: Infrastructure as Code
### Step-by-Step Modular Bicep Design, Validation, and Deployment Guide

</div>

---

## Purpose

This runbook documents how to translate the completed Azure AI platform into modular Bicep, separate parameters from implementation logic, validate changes, deploy into a clean scope, and document manual prerequisites.

Use the accompanying [Phase 9 Portfolio Case Study](../09-infrastructure-as-code.md) for architecture, decisions, results, and lessons learned.

---

## Scope

- Repository scaffolding
- Bicep modules
- Parameters
- RBAC as code
- Policy and monitoring
- Backup resources
- Sentinel resources
- Lighthouse onboarding package
- Build, lint, validate, what-if, and deployment
- Post-deployment verification

---

## Prerequisites

- Phases 1–8 complete or designed
- Azure CLI and Bicep CLI
- Git repository
- Test subscription/resource group
- Object IDs for pre-created Entra groups
- Permission to deploy at required scopes
- No secrets stored in source control

---

## Repository Structure

```text
bicep/
├── main.bicep
├── modules/
│   ├── network.bicep
│   ├── keyvault.bicep
│   ├── ai-services.bicep
│   ├── policy.bicep
│   ├── monitoring.bicep
│   ├── backup.bicep
│   └── sentinel.bicep
├── parameters/
│   ├── dev.bicepparam
│   └── prod.example.bicepparam
└── README.md

lighthouse/
├── main.bicep
├── customer-onboarding.bicepparam
└── README.md
```

---

# 1. Create the Repository Scaffold

Create the folders and empty files. Add:

- `.gitignore`
- No secret-bearing parameter files
- README describing deployment scopes
- Naming conventions
- Required manual prerequisites

---

# 2. Define Shared Parameters

Recommended parameters:

- `location`
- `environment`
- `namePrefix`
- Network CIDRs
- Group object IDs
- Log retention
- Backup retention
- Policy effects
- Model/SKU values
- Managing tenant ID for Lighthouse package

Use secure runtime inputs or Key Vault references for secrets.

---

# 3. Build `network.bicep`

Represent:

- Hub VNet
- Spoke VNet
- Subnets
- NSGs and associations
- VNet peering
- Bastion dependencies
- Private DNS zones and links
- Private-endpoint subnet policies

Export subnet and VNet IDs for downstream modules.

---

# 4. Build `keyvault.bicep`

Represent:

- Key Vault
- RBAC authorization
- Soft delete/purge-protection posture
- Private endpoint
- DNS zone group
- Optional key resource when supported and operationally appropriate

Do not store key material in code.

---

# 5. Build `ai-services.bicep`

Represent:

- Azure AI Services resource
- System-assigned identity
- Network restrictions
- Private endpoint
- DNS integration
- Supported model deployment configuration
- Diagnostic settings through a dependent monitoring module or extension resource

Document settings that remain Foundry/manual.

---

# 6. Build `policy.bicep`

Represent:

- Built-in and custom policy assignments
- Scope
- Parameters
- Audit/Deny effect selection
- Exclusions or exemptions only where intentionally required

Use Audit for development where Deny would block unfinished dependencies.

---

# 7. Build `monitoring.bicep`

Represent:

- Log Analytics Workspace
- Retention
- Diagnostic settings for supported resources
- Activity-log export where deployment scope permits
- Storage archival destination

Ensure resources exist before extension-resource deployment.

---

# 8. Build `backup.bicep`

Represent:

- Recovery Services Vault
- Backup policy
- Supported vault settings
- VM backup-protection resources where practical

Document any protection operation that requires a separate deployment step or API behavior.

---

# 9. Build `sentinel.bicep`

Represent:

- Sentinel onboarding solution
- Analytics rules where supported
- Automation rules
- Watchlists or workbooks where intentionally included

Store KQL in readable files or variables and preserve MITRE mappings.

---

# 10. Encode RBAC Assignments

Use:

- Existing Entra group object IDs as parameters
- Built-in role definition IDs
- Deterministic names, for example:

```bicep
name: guid(targetScope.id, principalId, roleDefinitionId)
```

Avoid Owner unless required and justified.

---

# 11. Build the Lighthouse Package

Keep Lighthouse separate because the managed customer deploys it.

Include:

- Managing tenant ID
- Authorized group object ID
- Role definition IDs
- Offer metadata
- Customer scope instructions
- Offboarding instructions

Do not automatically deploy customer acceptance from the managing tenant.

---

# 12. Build `main.bicep`

Orchestrate modules with explicit dependencies and outputs.

Typical order:

1. Network
2. Key Vault
3. Azure AI
4. Monitoring
5. Policy
6. Backup
7. Sentinel
8. RBAC extension resources

Use module outputs rather than reconstructing resource IDs manually.

---

# 13. Build and Lint

Run:

```bash
az bicep build --file bicep/main.bicep
```

Resolve:

- Syntax errors
- Linter warnings
- Unused parameters
- Hard-coded environment values
- Insecure defaults

---

# 14. Validate the Deployment

For resource-group scope:

```bash
az deployment group validate \
  --resource-group <test-rg> \
  --template-file bicep/main.bicep \
  --parameters bicep/parameters/dev.bicepparam
```

Use subscription deployment commands for subscription-scope resources.

---

# 15. Run What-If

```bash
az deployment group what-if \
  --resource-group <test-rg> \
  --template-file bicep/main.bicep \
  --parameters bicep/parameters/dev.bicepparam
```

Review creates, modifications, deletions, and ignored changes.

Do not deploy if unexpected destructive changes appear.

---

# 16. Deploy to a Clean Scope

1. Create a clean test resource group or subscription scope.
2. Deploy the template.
3. Capture deployment output.
4. Do not target the manually built environment first.
5. Record manual prerequisites and failures.

---

# 17. Validate the Deployment

Confirm:

- VNets/subnets/NSGs
- Private DNS
- Key Vault
- Azure AI identity and networking
- Log Analytics
- Diagnostic settings
- Policies
- Recovery Services Vault/policy
- Sentinel onboarding
- RBAC assignments

Run a second what-if or redeployment to evaluate idempotency.

---

# 18. Completion Checklist

## Code

- [ ] Modules created
- [ ] Parameters separated
- [ ] Secrets excluded
- [ ] Outputs used
- [ ] RBAC deterministic
- [ ] Policy effects parameterized
- [ ] Lighthouse separated

## Validation

- [ ] Bicep build passes
- [ ] Linter reviewed
- [ ] Deployment validation passes
- [ ] What-if reviewed
- [ ] Clean-scope deployment succeeds
- [ ] Post-deployment checks complete
- [ ] Redeployment behavior reviewed

## Documentation

- [ ] README created
- [ ] Required scopes documented
- [ ] Manual prerequisites documented
- [ ] Unsupported resources documented
- [ ] Cleanup documented
- [ ] Evidence captured

---

# 19. Troubleshooting

## Role assignment already exists

Use deterministic GUID naming and verify the principal, scope, and role combination.

## Diagnostic setting deployment fails

Ensure the target resource and workspace exist, use the correct extension-resource scope, and verify supported categories.

## Policy assignment fails at scope

Use the correct deployment scope, location for subscription deployments, managed identity when remediation requires it, and valid definition IDs.

## Private endpoint deployment fails

Check subnet policies, DNS zone IDs, target subresource names, and regional service support.

## Backup protection cannot deploy

Separate vault/policy creation from protection if required and validate supported API versions and workload state.

## Lighthouse deployment fails

Confirm the customer executes the deployment at the intended scope with correct managing tenant and principal IDs.

---

## Related Documentation

- [Phase 9 Portfolio Case Study](../09-infrastructure-as-code.md)
- [Phase 8 — Red Team Validation](../08-red-team-validation.md)
- [Project Overview](../../README.md)

---

<div align="center">

**Runbook complete — review what-if output and manual prerequisites before every environment deployment.**

</div>
