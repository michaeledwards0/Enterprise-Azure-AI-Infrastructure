<div align="center">

# Phase 3: Azure AI Services (OpenAI) Deployment
### Deploying the AI Workload — Contoso AI Labs

</div>

---

## Overview

With identity (Phase 1) and network isolation (Phase 2) in place, Phase 3 deploys the actual AI workload: an Azure AI services resource with the Azure OpenAI model deployment, connected exclusively through the private endpoint subnet built in Phase 2, encrypted with customer-managed keys instead of Microsoft-managed defaults, and gated by Guardrails at the deployment level so no model is ever live without a content policy attached. This is also the phase where Phase 1's identity groups stop being inert containers and become functional — Azure RBAC role assignments are applied here for the first time, connecting AI-Admins, AI-Developers, and AI-Users to real permissions on a real resource.

**Environment:** Personal Azure tenant (`contosoailabs.onmicrosoft.com`) | **Duration:** ~2-3 hours | **Standard:** SC-500 blueprint

**Business context:** the deployed model is framed as Contoso AI Labs' internal engineering assistant — intended for internal knowledge queries by employees — rather than a customer-facing product. This gives the security controls in this phase and the detections in Phase 5 a realistic purpose to defend, without requiring a custom application layer; all interaction happens through the Azure AI Foundry playground or direct API calls, consistent with the Path 1 scope decided at the start of this project.

> **Naming note:** Microsoft has rebranded the product family from "Cognitive Services" to **Azure AI services.** However, the built-in RBAC role names have not been renamed to match — roles like `Cognitive Services OpenAI Contributor` and `Cognitive Services OpenAI User` still carry the legacy name in the role catalog itself. This doc refers to the resource as Azure AI services throughout, but role assignment screenshots will show the legacy role names, since that's what Azure currently exposes.

---

## Case Study

### Objective
Deploy the Azure AI services (Azure OpenAI) workload into the isolated network from Phase 2, with no public network exposure, encryption keys under organizational control, and access permissions enforced through the identity groups established in Phase 1 — closing the loop between identity design and actual resource authorization.

### Approach

Phase 1 built the AI-Admins, AI-Developers, and AI-Users groups, but they had nothing to attach to yet — Azure RBAC assigns permissions to resources, and no resource existed until this phase. So the first real decision here was sequencing: deploy the resource, then immediately turn those groups from organizational containers into enforced permissions. AI-Admins got Contributor at the resource group level, scoped as Eligible through PIM rather than standing access, consistent with how the group was described back in Phase 1 as requiring just-in-time activation. AI-Developers and AI-Users were never scoped that way, so they got Active, standing roles instead — Cognitive Services OpenAI Contributor and User respectively — matching their Phase 1 descriptions of deploy/configure/test versus consume-only access.

For identity on the resource itself, I used a system-assigned managed identity instead of API keys. This lets the AI resource authenticate to Key Vault without any credential ever sitting in code or config — removing an entire class of credential-leak risk before it can exist.

Encryption was the part that took the most iteration. I moved off Microsoft-managed keys to customer-managed keys so key ownership sits in my own Key Vault, which matters because it means I control rotation and revocation independently of Microsoft rather than trusting a default I have no visibility into. I set the key rotation policy to 12 months — long enough to avoid unnecessary re-encryption overhead, short enough to satisfy a compliance-driven freshness requirement. Getting CMK actually configured meant working through several access boundaries in sequence, not a single toggle: temporarily opening the vault's network for provisioning, granting myself Key Vault Crypto Officer for key management, allowing my own client IP through the firewall for the key picker, and then manually granting the managed identity Key Vault Crypto Service Encryption User after the portal's promise to handle that automatically didn't hold. That last part was the real lesson — Key Vault's data-plane and management-plane access are genuinely separate systems, and "the portal says it will handle this" isn't something to take on faith without verifying it actually did. Once the key was live, I reverted the vault's network posture back down rather than leaving the exception open permanently.

Guardrails (Azure's current name for content filtering at the deployment level) were configured at creation rather than added later, so no model deployment in this environment is ever live without a policy attached. Since this deployment is framed as an internal engineering assistant for employees rather than a customer-facing product, I leaned toward protecting company and user data over maximizing creative flexibility: Jailbreak and Indirect Prompt Injection protections enabled and set to block, Content Harms (Hate, Sexual, Violence, Self-Harm) left at Medium blocking across input and output, Protected Materials enabled, and Sensitive Data Leakage's PII guardrail enabled with at least one data type selected — directly protecting the kind of internal knowledge queries this assistant is meant to handle.

Getting to the point where I could deploy a model at all also turned up a gap I hadn't expected: creating the Azure AI services resource doesn't create a Foundry Project — they're two separate layers, and the Project-creation flow has no option to attach to an existing resource, only to provision a new one. That auto-provisioned resource walked straight into this project's own subscription-scoped deny-public-access policy, which was the correct behavior, not a bug — so rather than weakening the policy, I scoped a narrow exemption to a dedicated, disposable resource group built solely to hold that scaffolding piece, keeping it fully separate from the actual protected workload.

On the model itself, the original plan was `gpt-4o-mini`, but it was retired by Azure mid-project, so I moved to `gpt-5-mini` as the equivalent-tier replacement. At roughly $0.25 per 1M input tokens and $2.00 per 1M output tokens, it runs about 5x cheaper than full-scale gpt-5 — the right tradeoff for a lab environment where the goal is demonstrating real deployment patterns under real budget constraints, not maximizing model capability. It was also a reminder that Azure's model catalog turns over on its own schedule, and a real production owner of this deployment would need a process for tracking retirement dates rather than treating a deployed model as permanent.

### Controls Implemented
- Azure AI services resource deployed with public network access disabled, reachable only via a private endpoint deployed into the Phase 2 `snet-private-endpoints` subnet
- System-assigned managed identity for Key Vault authentication
- Customer-managed keys for encryption at rest, stored in a dedicated Key Vault using the RBAC permission model, with data-plane access scoped separately to the admin account (key management) and the AI service's managed identity (wrap/unwrap operations only)
- Guardrails configured at the model deployment level (Jailbreak, Indirect Prompt Injection, Content Harms, Protected Materials, Sensitive Data Leakage/PII)
- Azure RBAC role assignments connecting Phase 1's identity groups to real permissions on the AI resource
- Private DNS zone groups created and linked to the spoke VNet, one per resource (`privatelink.vaultcore.azure.net` for Key Vault, `privatelink.cognitiveservices.azure.com` for Azure AI services)
- A disposable, out-of-architecture scaffolding resource used solely to satisfy the Foundry Project workspace layer, deliberately excluded from CMK, private endpoints, and the project's security narrative — kept isolated in its own resource group with a scoped policy exemption, distinct from the actual protected workload

### RBAC Role Assignments

| Group | Role | Scope | Assignment Type | Rationale |
|---|---|---|---|---|
| **AI-Admins** | `Contributor` | `rg-secure-ai-prod` resource group | Eligible (PIM, 3-month expiration, 4hr activation, MFA/justification/approval required) | Full administrative control to manage all resources in the environment, consistent with the group's Phase 1 "PIM required for activation" description |
| **AI-Developers** | `Cognitive Services OpenAI Contributor` | Azure AI services resource | Active (standing) | Deploy models, fine-tune, and generate content — matches the "deploy, configure, and test" scope defined in Phase 1 |
| **AI-Users** | `Cognitive Services OpenAI User` | Azure AI services resource | Active (standing) | Inference and content generation only, no deployment or configuration rights — matches "consume AI services via approved interfaces only" |

### Frameworks Applied
- Microsoft Cloud Adoption Framework — Data Encryption and Key Management
- OWASP Top 10 for LLM Applications — LLM06 (Sensitive Information Disclosure), addressed via network isolation and CMK
- CIS Microsoft Azure Foundations Benchmark v2.0 — Section 4 (Storage/Key Management analog applied to Cognitive/AI resources)
- NIST 800-207 (Zero Trust Architecture) — least-privilege authorization

### Evidence

| Control | What it proves | Screenshot |
|---|---|---|
| Key Vault deployed | `kv-contoso-ai` provisioned with public access disabled and connected via private endpoint | <img width="1267" height="527" alt="image" src="https://github.com/user-attachments/assets/3960e681-e418-452c-9d40-f2af970e49af" /> |
| Azure AI services resource deployed | `ai-contoso-openai` provisioned with public network access disabled and system-assigned managed identity enabled | <img width="1005" height="605" alt="image" src="https://github.com/user-attachments/assets/d3735a2a-0b86-409a-9034-2d5f3684f9e2" /> |
| CMK encryption configured | Encryption blade showing customer-managed key successfully saved, pointing to `kv-contoso-ai` / `key-contoso-ai-cmk`, plus the layered IAM role assignments (admin: Key Vault Crypto Officer; managed identity: Key Vault Crypto Service Encryption User) | <img width="1027" height="744" alt="image" src="https://github.com/user-attachments/assets/e3d5b089-c769-4935-923e-30b2d08cdffe" /> |
| Foundry Project connected | Scoped policy exemption for the disposable scaffolding resource group, and the Project's Connected Resources list showing `ai-contoso-openai` attached | <img width="1016" height="595" alt="image" src="https://github.com/user-attachments/assets/7491a3f9-4531-4568-b801-c0adfc7d113d" /> |
| Model deployed with Guardrails | Full Guardrails configuration (Jailbreak, Indirect Prompt Injection, Content Harms, Protected Materials, PII) and model deployment page confirming deployment against `ai-contoso-openai` | <img width="1305" height="878" alt="image" src="https://github.com/user-attachments/assets/3e1b1ecf-92d5-493a-94d9-3cebd62873a3" /> |
| RBAC role assignments | Access control (IAM) → Role assignments tab showing all three group-to-role mappings, AI-Admins as Eligible and the other two as Active | <img width="1328" height="782" alt="image" src="https://github.com/user-attachments/assets/03c12c8b-6e7a-4715-9902-c5ffe914cde8" /> |
| PIM activation requirements | Contributor role's PIM settings page showing 4hr max activation, MFA, justification, ticket info, and approval required | <img width="936" height="896" alt="image" src="https://github.com/user-attachments/assets/defbc663-632d-435a-9d50-ce3e02b8d3ba" /> |
| Test VM deployed | `vm-ai-workload-test` running in `snet-ai-workload` with no public IP | `screenshots/phase-03/05a-test-vm-deployed.png` |
| Private DNS resolution verified | PowerShell output from inside the VNet resolving `ai-contoso-openai`'s endpoint to a private `10.1.1.0/24` address | `screenshots/phase-03/05-private-dns-resolution.png` |

### Lessons Learned
*[Fill in after execution — the PIM-for-resources discovery is a strong candidate: extending Phase 1's PIM pattern to a resource-level Azure RBAC role surfaced that PIM for Azure resources lives in a completely separate part of the portal from a resource group's own IAM blade, requiring the resource group to be explicitly discovered and onboarded first. Also worth noting the fresh subscription's zero standing compute quota, which required a quota increase request before the first VM of the project — small and quickly approved, but a good example of a platform guardrail that has nothing to do with the architecture itself and shouldn't be mistaken for a configuration mistake.]*

---

## Next Phase

➡️ **[Phase 4: Governance & Defender for Cloud](./04-governance-defender.md)**

With the workload deployed, Phase 4 layers Defender for Cloud workload protection and an expanded governance baseline over everything built so far.

---

<details>
<summary><strong>📋 Full Execution Guide (click to expand)</strong> — step-by-step build instructions and completion checklist</summary>

<br>

[... execution guide content unchanged from your original — Sections 1 through 5, plus the Completion Checklist ...]

</details>

---

<div align="center">

[← Back to Project Overview](../README.md)

</div>
