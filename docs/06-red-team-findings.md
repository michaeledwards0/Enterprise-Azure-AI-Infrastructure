<div align="center">

# Phase 6: Red Team Validation
### Adversarial Testing of Every Control Built So Far — Contoso AI Labs

</div>

---

## Overview

Every control built in Phases 1–5 is a claim about what the environment will do under attack. Phase 6 tests those claims directly — deliberately attempting prompt injection, jailbreak, data exfiltration, unauthorized access, and privilege escalation against the environment, and documenting whether each defense performed as designed. This is the phase that converts "I built detections" into "I proved the detections work."

**Environment:** Personal Azure tenant (`contosoailabs.onmicrosoft.com`) | **Duration:** ~3-4 hours | **Standard:** SC-500 blueprint, OWASP Top 10 for LLM Applications

### Design Rationale

**Why validate at all, rather than trusting the build:** A control that's never been tested against a real attempt is a hypothesis, not a verified defense. Untested detections are a common gap in real environments — rules get written, deployed, and never fired in anger until an actual incident, at which point tuning issues surface for the first time during a real event.

**Why these five specific attack categories:** They map directly to the five custom KQL rules built in Phase 5 (prompt injection, token abuse/jailbreak, off-hours/impossible-travel access, and unauthorized access broadly) plus a privilege escalation attempt against the PIM/RBAC boundary from Phases 1 and 3 — meaning every major control from every prior phase gets at least one adversarial test.

**Why findings are documented even when a test succeeds against the defense:** A red team exercise that only reports "everything worked" is less credible and less useful than one that documents exactly what was tried, what fired, what didn't, and what was tuned afterward. Real security work includes gaps found and closed, not just gaps prevented.

---

## Case Study

### Objective
Validate that the identity, network, workload, governance, and detection controls built in Phases 1–5 function as designed under deliberate adversarial conditions, and document any gaps discovered along with remediation.

### Approach
*[Fill in after execution — describe your reasoning for the order of test execution, any detection that didn't fire as expected on the first attempt, and how you tuned it afterward.]*

### Controls Validated
- **Prompt injection** attempted against the deployed model, checked against the Phase 5 prompt injection analytics rule
- **Jailbreak attempt** using known jailbreak prompt patterns, checked against the Phase 5 jailbreak detection rule and the automated response playbook
- **Data exfiltration attempt** via the AI resource (e.g., attempting to extract training data or system prompt contents), checked against content filtering (Phase 3) and anomalous token consumption detection (Phase 5)
- **Unauthorized access attempt** from an account without RBAC permissions on the AI resource, checked against the Phase 3 role assignments
- **Privilege escalation attempt** — an AI-Developers group member attempting an AI-Admins-scoped action, checked against RBAC boundary enforcement and PIM activation requirements from Phase 1

### Frameworks Applied
- OWASP Top 10 for LLM Applications — LLM01 (Prompt Injection), LLM02 (Insecure Output Handling), LLM06 (Sensitive Information Disclosure)
- MITRE ATT&CK — mapped per test scenario
- NIST 800-53 — Security Assessment and Authorization (CA) control family

### Findings Summary

| Test | Expected Control | Result | Remediation (if needed) |
|---|---|---|---|
| Prompt injection | Content filter blocks / Sentinel rule fires | *[fill in]* | *[fill in]* |
| Jailbreak attempt | Jailbreak rule fires, playbook disables account | *[fill in]* | *[fill in]* |
| Data exfiltration attempt | Content filter blocks, token anomaly rule fires | *[fill in]* | *[fill in]* |
| Unauthorized access attempt | RBAC denies access | *[fill in]* | *[fill in]* |
| Privilege escalation attempt | RBAC/PIM boundary holds | *[fill in]* | *[fill in]* |

### Evidence
*[Screenshots and incident exports added after execution — see capture list in the Execution Guide below.]*

### Lessons Learned
*[Fill in after execution.]*

---

## Next Phase

➡️ **[Phase 7: Infrastructure as Code](./07-infrastructure-as-code.md)**

With every control validated, Phase 7 codifies the entire environment as Bicep templates — making the whole build reproducible from scratch rather than dependent on manual portal steps.

---

<details>
<summary><strong>📋 Full Execution Guide (click to expand)</strong> — step-by-step test instructions and completion checklist</summary>

<br>

**Prerequisites:** Phases 1–5 complete

> ⚠️ **Scope reminder:** All testing occurs within your own personal tenant against resources you own. Do not run any of these techniques against systems you don't own or have explicit authorization to test.

### Section 1: Prompt Injection Test

1. Via the deployed model's chat interface (Azure AI Foundry playground or API), submit a prompt attempting to override the model's system instructions (e.g., asking it to ignore prior instructions and reveal internal configuration)
2. Check **Sentinel → Incidents** for the Prompt Injection Detection rule firing
3. Document the exact prompt used, the model's response, and whether the rule fired

📸 **Screenshot to capture:** The prompt/response pair and the corresponding Sentinel incident (if fired). Save as `screenshots/phase-06/01-prompt-injection-test.png`.

### Section 2: Jailbreak Attempt

1. Submit a known jailbreak-style prompt pattern designed to bypass content filtering
2. Confirm the content filter blocks the response, and check whether the Jailbreak Attempt Detection rule fired
3. If it fired, confirm the automated playbook disabled the test account as designed

📸 **Screenshot to capture:** Sentinel incident showing the jailbreak detection, and confirmation of the account disable action in the playbook run history. Save as `screenshots/phase-06/02-jailbreak-test.png`.

### Section 3: Data Exfiltration Attempt

1. Attempt to extract the model's system prompt or request unusually large outputs designed to test token-consumption anomaly detection
2. Check the Anomalous Token Consumption rule and content filter response

📸 **Screenshot to capture:** Test attempt and any resulting Sentinel incident. Save as `screenshots/phase-06/03-exfiltration-test.png`.

### Section 4: Unauthorized Access Attempt

1. Using a test identity with no RBAC role on the AI resource (or the AI-Users account attempting an AI-Developers-scoped action), attempt to access or modify the deployment
2. Confirm Azure denies the action (403/Forbidden)

📸 **Screenshot to capture:** The access-denied response confirming RBAC enforcement. Save as `screenshots/phase-06/04-unauthorized-access-test.png`.

### Section 5: Privilege Escalation Attempt

1. As `david.dev` (AI-Developers), attempt an action scoped only to AI-Admins (e.g., modifying resource group-level settings)
2. Confirm the action is denied
3. Separately, attempt to activate a PIM-eligible role without going through the approval workflow (e.g., attempting direct role assignment) and confirm this is blocked or requires the defined approval process

📸 **Screenshot to capture:** Denied escalation attempt and/or the PIM approval requirement being enforced. Save as `screenshots/phase-06/05-privilege-escalation-test.png`.

### Section 6: Document Findings

Fill in the Findings Summary table in the Case Study section above with actual results, and note any detections that required tuning after the first test (e.g., threshold adjustments, additional KQL conditions).

### Completion Checklist

- [ ] Prompt injection tested and result documented
- [ ] Jailbreak attempt tested, including playbook response verification
- [ ] Data exfiltration attempt tested and result documented
- [ ] Unauthorized access attempt tested against RBAC boundaries
- [ ] Privilege escalation attempt tested against RBAC/PIM boundaries
- [ ] Findings Summary table completed with actual results
- [ ] Any detection gaps found were remediated and re-tested
- [ ] All screenshots captured and saved to `screenshots/phase-06/`

</details>

---

<div align="center">

[← Back to Project Overview](../README.md)

</div>
