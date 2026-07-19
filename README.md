<div align="center">

# Enterprise Azure AI Infrastructure

### End-to-End Cloud Security Architecture for Azure AI Workloads

*A Zero Trust reference implementation demonstrating how to design, deploy, govern, monitor, recover, operate, validate, and codify a production-style AI workload on Microsoft Azure.*

**Environment:** Personal Azure Tenant  
**Author:** Michael Edwards — Cloud Security Engineer

</div>

---

## Executive Summary

**Contoso AI Labs** is a fictional startup deploying an internal Azure AI workload into a production-style Microsoft Azure environment.

As the Cloud Security Engineer, I designed and implemented the platform across nine phases:

1. **Establish** trusted identities and privileged-access controls
2. **Isolate** the workload through private networking
3. **Deploy** the Azure AI service with managed identity, private endpoints, CMK encryption, and guardrails
4. **Govern and protect** the environment with Azure Policy and Microsoft Defender for Cloud
5. **Detect and respond** with Microsoft Sentinel, KQL analytics rules, and controlled automation
6. **Recover** stateful workloads through Azure Backup and restore validation
7. **Operate across tenants** using Azure Lighthouse and delegated administration
8. **Validate controls** through authorized attack simulation and findings management
9. **Codify** the platform through modular Bicep Infrastructure as Code

This repository documents the complete engineering lifecycle—from identity and network design through governance, detection, recovery, multi-tenant operations, red-team validation, and Infrastructure as Code handoff.

> **Project outcome:** A privately connected, identity-driven, monitored, recoverable, and reproducible Azure AI platform designed around Zero Trust and enterprise cloud-security practices.

---

## Architecture Overview

```text
┌──────────────────────────────────────────────────────────────────────┐
│                         IDENTITY LAYER                               │
│ Entra ID · MFA · Conditional Access · PIM · Break-Glass · RBAC      │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                      NETWORK ISOLATION LAYER                         │
│ Hub-Spoke VNets · Private Endpoints · Private DNS · NSGs · Bastion  │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                         AI WORKLOAD LAYER                            │
│ Azure AI Services · Managed Identity · CMK · Guardrails · Foundry   │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                    GOVERNANCE & PROTECTION LAYER                     │
│ Azure Policy · Defender for Cloud · Secure Score · Compliance       │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│               MONITORING, DETECTION & RESPONSE LAYER                │
│ Log Analytics · Azure Monitor · Sentinel · KQL · Logic Apps         │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│              BUSINESS CONTINUITY & RECOVERY LAYER                   │
│ Recovery Services Vault · Azure Backup · Recovery Points · Restore  │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                 MULTI-TENANT OPERATIONS LAYER                       │
│ Azure Lighthouse · Delegated Resource Management · Cross-Tenant RBAC│
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                    RED TEAM VALIDATION LAYER                        │
│ Prompt Injection · Jailbreak · Access Testing · Findings · Retest   │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────┐
│                 INFRASTRUCTURE AS CODE LAYER                        │
│ Modular Bicep · Parameters · Policy as Code · Deployment Validation │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Project Phases

| Phase | Focus Area | Status |
|:---:|---|:---:|
| **[Phase 1](./docs/01-identity-fortress.md)** | Identity Fortress — Entra ID, Conditional Access, PIM, break-glass access, and baseline governance | 🟢 Complete |
| **[Phase 2](./docs/02-network-architecture.md)** | Network Architecture & Isolation — hub-spoke VNets, NSGs, Bastion, private endpoints, and private DNS | 🟢 Complete |
| **[Phase 3](./docs/03-openai-deployment.md)** | Azure AI Services Deployment — managed identity, CMK, Key Vault, guardrails, RBAC, and private validation | 🟢 Complete |
| **[Phase 4](./docs/04-governance-defender.md)** | Governance & Defender for Cloud — policy compliance, Secure Score, workload protection, and logging readiness | 🟢 Complete |
| **[Phase 5](./docs/05-detection-engineering.md)** | Detection Engineering — Sentinel, KQL analytics, entity mappings, incidents, and controlled automation | ⚪ Planned / In Progress |
| **[Phase 6](./docs/06-business-continuity-recovery.md)** | Business Continuity & Recovery — Recovery Services Vault, VM backup, recovery points, and restore validation | ⚪ Planned |
| **[Phase 7](./docs/07-multi-tenant-administration.md)** | Multi-Tenant Administration — Azure Lighthouse, delegated operations, cross-tenant RBAC, and offboarding | ⚪ Planned |
| **[Phase 8](./docs/08-red-team-validation.md)** | Red Team Validation — authorized attack simulation, detection validation, findings, remediation, and retesting | ⚪ Planned |
| **[Phase 9](./docs/09-infrastructure-as-code.md)** | Infrastructure as Code — modular Bicep, parameters, policy as code, validation, and deployment automation | ⚪ Planned |

---

## Repository Structure

```text
secure-ai-deployment-azure/
├── README.md
├── docs/
│   ├── 01-identity-fortress.md
│   ├── 02-network-architecture.md
│   ├── 03-openai-deployment.md
│   ├── 04-governance-defender.md
│   ├── 05-detection-engineering.md
│   ├── 06-business-continuity-recovery.md
│   ├── 07-multi-tenant-administration.md
│   ├── 08-red-team-validation.md
│   └── 09-infrastructure-as-code.md
├── runbooks/
│   ├── 01-identity-fortress-runbook.md
│   ├── 02-network-architecture-runbook.md
│   ├── 03-openai-deployment-runbook.md
│   ├── 04-governance-defender-runbook.md
│   ├── 05-detection-engineering-runbook.md
│   ├── 06-business-continuity-recovery-runbook.md
│   ├── 07-multi-tenant-administration-runbook.md
│   ├── 08-red-team-validation-runbook.md
│   └── 09-infrastructure-as-code-runbook.md
├── bicep/
│   ├── main.bicep
│   ├── modules/
│   │   ├── identity.bicep
│   │   ├── network.bicep
│   │   ├── keyvault.bicep
│   │   ├── ai-services.bicep
│   │   ├── policy.bicep
│   │   ├── monitoring.bicep
│   │   ├── backup.bicep
│   │   └── sentinel.bicep
│   └── parameters/
│       ├── dev.bicepparam
│       └── prod.example.bicepparam
├── lighthouse/
│   ├── main.bicep
│   ├── customer-onboarding.bicepparam
│   └── README.md
├── kql/
│   ├── prompt-injection-detection.kql
│   ├── jailbreak-attempts.kql
│   ├── anomalous-token-consumption.kql
│   ├── off-hours-ai-access.kql
│   ├── impossible-travel-ai.kql
│   └── break-glass-signin.kql
├── policies/
│   ├── deny-public-ip.json
│   ├── require-private-endpoints.json
│   ├── require-managed-identity.json
│   └── require-diagnostic-settings.json
├── playbooks/
│   └── auto-contain-ai-test-user.json
├── diagrams/
│   ├── architecture-overview.png
│   ├── network-topology.png
│   ├── detection-pipeline.png
│   ├── backup-recovery-flow.png
│   └── lighthouse-delegation.png
└── screenshots/
    ├── phase-01/
    ├── phase-02/
    ├── phase-03/
    ├── phase-04/
    ├── phase-05/
    ├── phase-06/
    ├── phase-07/
    ├── phase-08/
    └── phase-09/
```

---

## Security Frameworks Applied

- **NIST SP 800-207** — Zero Trust Architecture
- **NIST SP 800-53** — Security, privacy, configuration, access-control, monitoring, and contingency controls
- **NIST SP 800-61** — Incident detection, analysis, containment, and response
- **NIST SP 800-34** — Contingency planning and recovery validation
- **NIST AI Risk Management Framework**
- **CIS Microsoft Azure Foundations Benchmark**
- **Microsoft Cloud Adoption Framework**
- **Microsoft Well-Architected Framework**
- **OWASP Top 10 for LLM Applications**
- **MITRE ATT&CK**

---

## Tools & Technologies

| Category | Technologies |
|---|---|
| **Identity** | Microsoft Entra ID, Conditional Access, PIM, Azure RBAC, managed identities |
| **Networking** | Azure VNets, VNet peering, private endpoints, Private DNS, NSGs, Azure Bastion |
| **AI workload** | Azure AI Services, Azure AI Foundry, model guardrails, content safety |
| **Encryption** | Azure Key Vault, customer-managed keys |
| **Governance** | Azure Policy, Defender for Cloud, Secure Score, compliance reporting |
| **Monitoring** | Azure Monitor, Log Analytics, diagnostic settings, Azure Activity Logs |
| **SIEM/SOAR** | Microsoft Sentinel, KQL, Logic Apps, analytics rules, entity mappings |
| **Recovery** | Recovery Services Vault, Azure Backup, restore validation |
| **Multi-tenant operations** | Azure Lighthouse, delegated resource management, cross-tenant RBAC |
| **Validation** | Authorized attack simulation, findings management, remediation, retesting |
| **Infrastructure as Code** | Bicep, ARM deployments, Azure CLI, Git |

---

## Engineering Outcomes

This project demonstrates the ability to:

- Design a Zero Trust Azure architecture
- Build hub-spoke networking with private service access
- Secure Azure AI Services with managed identity, Key Vault, CMK, RBAC, and guardrails
- Apply policy-driven governance and Defender for Cloud posture management
- Centralize Azure, identity, and AI telemetry
- Engineer Microsoft Sentinel detections and controlled response automation
- Design backup and restore procedures for recoverable workloads
- Delegate Azure resource operations securely across tenants
- Validate preventive and detective controls through authorized testing
- Convert a manual implementation into modular, version-controlled Bicep

---

## Key Learnings

- Azure security controls must be validated in their effective state rather than assumed from portal configuration.
- Management-plane access and data-plane access are separate and require different RBAC assignments.
- Policy enforcement should match platform maturity; Audit mode can be appropriate while dependencies are still being validated.
- Detection engineering begins with reliable telemetry, not with writing KQL.
- Backup is incomplete until a restore has been successfully tested.
- Azure Lighthouse delegates resource management while preserving customer ownership.
- Security findings are most valuable when remediation is followed by an exact retest.
- Infrastructure as Code makes architecture repeatable, but does not replace sound design decisions.

---

## Documentation Model

Each phase includes:

- A portfolio-ready case study
- Business and security context
- Architecture diagram
- Implementation summary
- Engineering decisions and tradeoffs
- Issues and resolutions
- Results and validation
- Evidence mapping
- Framework alignment
- Lessons learned
- A separate detailed implementation runbook

---

<div align="center">

*Michael Edwards · Cloud Security Engineer · Houston, TX*  
[GitHub](https://github.com/michaeledwards0) · [LinkedIn](https://linkedin.com/in/edwardsmichela)

</div>
