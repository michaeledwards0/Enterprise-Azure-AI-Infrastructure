<div align="center">

# Phase 7: Infrastructure as Code
### Codifying the Full Environment in Bicep — Contoso AI Labs

</div>

---

## Overview

Phases 1–6 were built manually through the Azure portal, deliberately, so each control could be understood and validated step by step. Phase 7 closes the project by codifying the entire environment as **modular Bicep templates** — meaning everything built by hand across six phases could be destroyed and redeployed identically from code alone. This is also where the Azure RBAC role assignments from Phase 3 become code rather than portal clicks, and where the Phase 5 detection layer becomes reproducible infrastructure.

**Environment:** Personal Azure tenant (`contosoailabs.onmicrosoft.com`) | **Duration:** ~4-5 hours | **Standard:** SC-500/AZ-104 blueprint

### Design Rationale

**Why IaC as the final phase rather than the first:** Building manually first meant every decision in Phases 1–6 could be reasoned through and validated interactively before being locked into code. Doing IaC first would have meant encoding assumptions before they'd been tested against a real environment.

**Why modular Bicep instead of one large template:** Splitting into `identity.bicep`, `network.bicep`, `openai.bicep`, `keyvault.bicep`, `policy.bicep`, and `sentinel.bicep` modules mirrors how a real platform team would structure ownership — different modules can be updated independently, reviewed independently, and reused across future projects without dragging the whole template along.

**Why RBAC role assignments specifically belong in code by this phase:** Manual role assignment (Phase 3) is fine for a one-time build, but it's not reproducible or auditable at scale. Encoding `Microsoft.Authorization/roleAssignments` as Bicep resources means the identity-to-permission mapping is version-controlled, reviewable in a pull request, and redeployable exactly, rather than dependent on someone remembering to click the same three role assignments again.

---

## Case Study

### Objective
Codify the complete Contoso AI Labs environment — identity, network, workload, governance, and detection — as modular, parameterized Bicep templates capable of redeploying the entire environment from a clean subscription.

### Approach
*[Fill in after execution — describe any resources that couldn't be fully codified (e.g., break-glass account password handling, which should remain a manual/offline process even in an IaC-driven environment), and your reasoning for parameter file structure.]*

### Controls Implemented
- `main.bicep` orchestrating all modules with a shared parameters file
- `identity.bicep` — security groups and RBAC role assignments (Phase 1 groups + Phase 3 role mappings)
- `network.bicep` — hub-spoke VNets, subnets, NSGs, Bastion, private DNS zone (Phase 2)
- `keyvault.bicep` — Key Vault with private endpoint and access policies (Phase 3)
- `openai.bicep` — Azure AI services resource, model deployment, content filter, private endpoint (Phase 3)
- `policy.bicep` — all governance policy assignments across Phases 1, 2, and 4
- `sentinel.bicep` — Log Analytics workspace, Sentinel onboarding, diagnostic settings, analytics rules (Phase 5)
- Validated end-to-end redeployment into a clean resource group

### What Remains Manual (Deliberately)
- Break-glass account password generation and offline storage — this should never be encoded in a template or stored in source control under any circumstance
- PIM approval workflow configuration — a one-time governance decision rather than a redeployable resource
- Initial admin consent/tenant-level setup

### Frameworks Applied
- Microsoft Cloud Adoption Framework — Infrastructure as Code discipline
- NIST 800-53 — Configuration Management (CM) control family

### Evidence
*[Screenshots and deployment output added after execution — see capture list in the Execution Guide below.]*

### Lessons Learned
*[Fill in after execution.]*

---

## Project Complete

This closes the seven-phase Secure AI Deployment on Azure build. Return to the **[master README](../README.md)** for the full architecture overview and phase index.

---

<details>
<summary><strong>📋 Full Execution Guide (click to expand)</strong> — step-by-step build instructions and completion checklist</summary>

<br>

**Prerequisites:** Phases 1–6 complete; Bicep CLI installed locally or Azure Cloud Shell available

### Section 1: Scaffold the Repository Structure

Confirm (or create) the following structure, matching the master README:

```
bicep/
├── main.bicep
├── modules/
│   ├── identity.bicep
│   ├── network.bicep
│   ├── openai.bicep
│   ├── keyvault.bicep
│   ├── policy.bicep
│   └── sentinel.bicep
└── parameters/
    └── main.parameters.json
```

### Section 2: Build `identity.bicep`

Encode:
- The three security groups (AI-Admins, AI-Developers, AI-Users) — note: Microsoft Graph Bicep support for Entra ID groups is limited; groups may need to be pre-created and referenced by object ID rather than created by Bicep directly
- RBAC role assignments as `Microsoft.Authorization/roleAssignments` resources, parameterized by group object ID and target scope, reproducing the Phase 3 assignments (Contributor for AI-Admins, Cognitive Services OpenAI Contributor for AI-Developers, Cognitive Services OpenAI User for AI-Users)

### Section 3: Build `network.bicep`

Encode the Phase 2 hub-spoke topology: both VNets, all subnets with correct CIDR ranges, NSGs with their rules, Bastion, and the private DNS zone with its virtual network link.

### Section 4: Build `keyvault.bicep` and `openai.bicep`

Encode the Phase 3 Key Vault (private endpoint, CMK-ready configuration) and the Azure AI services resource (managed identity, private endpoint, model deployment, content filter configuration).

### Section 5: Build `policy.bicep`

Encode all policy assignments from Phases 1, 2, and 4 as `Microsoft.Authorization/policyAssignments` resources, parameterized by effect (Deny/Audit) so they can be tuned per environment without editing the module itself.

### Section 6: Build `sentinel.bicep`

Encode the Log Analytics workspace, Sentinel onboarding, diagnostic settings for Entra ID and the AI resource, and the analytics rules from Phase 5 as `Microsoft.SecurityInsights/alertRules` resources.

### Section 7: Build `main.bicep` and Parameters

1. Orchestrate all modules in `main.bicep` with appropriate module dependencies (network before openai/keyvault, identity before policy role assignments, etc.)
2. Populate `parameters/main.parameters.json` with environment-specific values (region, naming prefixes, group object IDs)

### Section 8: Validate and Test Deployment

1. `az deployment group validate --resource-group <test-rg> --template-file main.bicep --parameters @parameters/main.parameters.json`
2. Deploy into a **separate, clean test resource group** — not `rg-secure-ai-prod` — to confirm the templates work without depending on manually-created resources
3. Document any manual prerequisite steps required before the Bicep deployment succeeds (e.g., pre-existing Entra groups)

📸 **Screenshot to capture:** Successful deployment output in the Azure Portal or CLI, showing all resources created from the template. Save as `screenshots/phase-07/01-bicep-deployment-success.png`.

### Section 9: Document and Finalize

1. Add a `README.md` inside the `bicep/` folder explaining module structure, parameters, and what remains manual
2. Update the master project README's Key Learnings section with a summary of the full seven-phase build

### Completion Checklist

- [ ] All six Bicep modules built (identity, network, openai, keyvault, policy, sentinel)
- [ ] `main.bicep` orchestrates all modules with correct dependencies
- [ ] Parameters file populated and environment-agnostic
- [ ] RBAC role assignments encoded as code, matching Phase 3
- [ ] Template validated successfully
- [ ] Test deployment into a clean resource group succeeded
- [ ] Manual/non-codified steps explicitly documented
- [ ] Master README updated with final project summary
- [ ] All screenshots captured and saved to `screenshots/phase-07/`

</details>

---

<div align="center">

[← Back to Project Overview](../README.md)

</div>
