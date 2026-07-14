<div align="center">

# Phase 5: Detection Engineering
### Microsoft Sentinel, Analytics Rules, and Automated Response — Contoso AI Labs

</div>

---

## Overview

Phase 5 builds the detection and response layer: a Log Analytics workspace, Microsoft Sentinel enabled on top of it, custom KQL analytics rules covering both identity and AI-specific threats, and a Logic App playbook for automated response. This phase also **retroactively enables the Conditional Access Insights and Reporting workbook** flagged as unavailable back in Phase 1 — once diagnostic settings stream Entra sign-in data into this workspace, that page comes online.

**Environment:** Personal Azure tenant (`contosoailabs.onmicrosoft.com`) | **Duration:** ~3-4 hours | **Standard:** SC-500 blueprint

### Design Rationale

**Why Sentinel instead of relying on Defender for Cloud alerts alone:** Defender for Cloud surfaces posture and known threat signatures. Sentinel is a SIEM — it correlates across data sources with custom logic you define, which is what makes detections like "impossible travel combined with AI resource access" possible. Defender tells you a resource is misconfigured; Sentinel tells you someone is actively behaving suspiciously across multiple signals.

**Why the break-glass monitoring alert specifically:** Phase 1 committed to monitoring any break-glass sign-in as a serious alert. Since that account should never be used outside a true emergency, any sign-in at all is inherently suspicious — this is one of the highest-confidence, lowest-false-positive detections possible in the entire environment.

**Why a Logic App playbook instead of manual response:** Manual response introduces delay between detection and containment. An automated playbook (e.g., disabling a user account on a jailbreak-attempt detection) closes that gap, though it's deliberately scoped to a narrow, high-confidence trigger to avoid automating a response to a false positive.

---

## Case Study

### Objective
Establish continuous detection coverage across identity and AI workload telemetry, enabling both custom threat detection and the sign-in analytics that were dependent on this phase's infrastructure since Phase 1.

### Approach
*[Fill in after execution — describe your reasoning for which KQL detections were prioritized first, any false positives encountered while tuning thresholds, and why the playbook trigger was scoped the way it was.]*

### Controls Implemented
- Log Analytics workspace deployed, with diagnostic settings streaming Entra ID `SignInLogs` and `AuditLogs`, plus Azure AI services resource logs
- Microsoft Sentinel enabled on the workspace
- Five custom KQL analytics rules: prompt injection detection, anomalous token consumption, off-hours AI access, impossible travel for AI resource access, jailbreak attempt detection
- Dedicated high-priority analytics rule for any break-glass account sign-in
- Logic App playbook for automated response to confirmed jailbreak attempts
- Conditional Access Insights and Reporting workbook now functional (dependency resolved from Phase 1)

### Frameworks Applied
- NIST 800-61 (Computer Security Incident Handling)
- OWASP Top 10 for LLM Applications — LLM01 (Prompt Injection), LLM04 (Model Denial of Service via token abuse)
- MITRE ATT&CK — mapped per analytics rule (see `kql/` folder for per-query mappings)

### Evidence
*[Screenshots added after execution — see capture list in the Execution Guide below.]*

### Lessons Learned
*[Fill in after execution.]*

---

## Next Phase

➡️ **[Phase 6: Red Team Validation](./06-red-team-findings.md)**

With detections live, Phase 6 validates them — deliberately triggering each attack pattern the analytics rules are designed to catch, and documenting whether they fired as expected.

---

<details>
<summary><strong>📋 Full Execution Guide (click to expand)</strong> — step-by-step build instructions and completion checklist</summary>

<br>

**Prerequisites:** Phases 1–4 complete

### Section 1: Deploy the Log Analytics Workspace

1. Azure Portal → **Log Analytics workspaces** → **+ Create**
   - Resource group: `rg-secure-ai-prod`
   - Name: `log-contoso-ai-sentinel`
   - Region: consistent with prior phases
2. Create

### Section 2: Enable Microsoft Sentinel

1. Azure Portal → search **Microsoft Sentinel** → **+ Create**
2. Select `log-contoso-ai-sentinel` as the workspace → **Add**

📸 **Screenshot to capture:** Sentinel overview page showing the workspace connected. Save as `screenshots/phase-05/01-sentinel-enabled.png`.

### Section 3: Connect Diagnostic Settings

1. Entra ID → **Diagnostic settings** → **+ Add diagnostic setting**
   - Name: `diag-entra-to-sentinel`
   - Logs: `SignInLogs`, `AuditLogs`, `NonInteractiveUserSignInLogs`
   - Destination: send to `log-contoso-ai-sentinel`
2. Azure AI services resource (`ai-contoso-openai`) → **Diagnostic settings** → **+ Add diagnostic setting**
   - Logs: all available categories (audit, request/response tracing if enabled)
   - Destination: same workspace

📸 **Screenshot to capture:** Diagnostic settings pages for both Entra ID and the AI resource, confirming both stream to the workspace. Save as `screenshots/phase-05/02-diagnostic-settings-connected.png`.

### Section 4: Verify the Insights and Reporting Workbook

1. Entra ID → **Conditional Access** → **Insights and reporting**
2. Confirm the page now loads (previously 401'd in Phase 1 due to the missing workspace)
3. Review policy impact trends across CA01–CA06

📸 **Screenshot to capture:** Insights and reporting workbook now loading successfully. Save as `screenshots/phase-05/03-insights-workbook-online.png`.

### Section 5: Build Custom Analytics Rules

Use the KQL queries in this repo's `kql/` folder as the query logic for each rule. For each: Sentinel → **Analytics** → **+ Create** → **Scheduled query rule**.

1. **Prompt Injection Detection** (`kql/prompt-injection-detection.kql`) — MITRE ATT&CK: Initial Access (T1190 analog for LLM context)
2. **Anomalous Token Consumption** (`kql/anomalous-token-consumption.kql`) — detects usage spikes suggesting abuse or a compromised key
3. **Off-Hours AI Access** (`kql/off-hours-ai-access.kql`) — flags AI resource access outside expected business hours
4. **Impossible Travel for AI Access** (`kql/impossible-travel-ai.kql`) — correlates sign-in geolocation with AI resource access timing
5. **Jailbreak Attempt Detection** (`kql/jailbreak-attempts.kql`) — pattern-matches known jailbreak prompt structures

**Additionally, build a dedicated high-severity rule:**

6. **Break-Glass Account Sign-In** — any successful or attempted sign-in from `breakglass@contosoailabs.onmicrosoft.com`, severity: **High**, since this account should never be used outside a genuine emergency

📸 **Screenshot to capture:** Analytics rules list showing all six rules active. Save as `screenshots/phase-05/04-analytics-rules-active.png`.

### Section 6: Build the Automated Response Playbook

1. Sentinel → **Automation** → **+ Create** → **Playbook with incident trigger**
2. Name: `auto-disable-jailbreak-user` (matches `playbooks/auto-disable-jailbreak-user.json` in this repo)
3. Logic: on trigger from the Jailbreak Attempt Detection rule, disable the associated user account in Entra ID and post a notification to your email
4. Attach the playbook to the Jailbreak Attempt Detection analytics rule as an automated response

📸 **Screenshot to capture:** Logic App designer showing the playbook logic, and the analytics rule's automation tab showing it attached. Save as `screenshots/phase-05/05-automation-playbook.png`.

### Section 7: Test End-to-End

1. Generate a benign test event that should trigger one rule (e.g., a deliberate off-hours sign-in or a break-glass test sign-in — coordinate timing so this doesn't trigger unrelated alerts)
2. Confirm an incident appears in **Sentinel → Incidents**
3. For the jailbreak rule specifically, confirm the playbook fired and the test account was disabled

📸 **Screenshot to capture:** Sentinel Incidents page showing a triggered incident from your test. Save as `screenshots/phase-05/06-incident-triggered.png`.

### Completion Checklist

- [ ] Log Analytics workspace deployed
- [ ] Microsoft Sentinel enabled on the workspace
- [ ] Diagnostic settings connected for Entra ID and the Azure AI services resource
- [ ] Conditional Access Insights and Reporting workbook confirmed functional
- [ ] Five custom KQL analytics rules built and active
- [ ] Break-glass account sign-in rule built as a dedicated high-severity detection
- [ ] Automated response playbook built and attached to the jailbreak detection rule
- [ ] End-to-end test confirms incident generation and (where applicable) automated response
- [ ] All screenshots captured and saved to `screenshots/phase-05/`

</details>

---

<div align="center">

[← Back to Project Overview](../README.md)

</div>
