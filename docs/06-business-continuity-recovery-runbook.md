<div align="center">

# Phase 6 Runbook: Business Continuity & Recovery
### Step-by-Step Azure Backup and Restore Validation Guide

</div>

---

## Purpose

This runbook documents how to deploy a Recovery Services Vault, configure a VM backup policy, protect the Phase 3 test VM, create a recovery point, validate restoration, and document recovery boundaries for managed Azure services.

Use the accompanying [Phase 6 Portfolio Case Study](../06-business-continuity-recovery.md) for the architecture, tradeoffs, results, and lessons learned.

---

## Scope

- Recovery Services Vault
- Vault security settings
- Daily VM backup policy
- VM backup enablement
- On-demand backup
- Job monitoring
- Alternate restore validation
- Recovery documentation
- Cleanup considerations

---

## Prerequisites

- Phases 1–5 complete
- `vm-ai-workload-test` exists
- VM and vault are in a supported region/subscription configuration
- Permissions to create vaults, policies, backups, and restore resources
- Temporary resource-group capacity for restore testing

---

## Resource Naming

| Resource | Name |
|---|---|
| Recovery Services Vault | `rsv-contoso-ai` |
| Backup policy | `bp-contoso-ai-daily` |
| Protected VM | `vm-ai-workload-test` |
| Restore resource group | `rg-contoso-ai-restore-test` |
| Restored disk or VM | `restore-vm-ai-workload-test` |

---

# 1. Define Recovery Requirements

Document:

- Protected workload
- Recovery owner
- Maximum acceptable data loss for the lab
- Maximum acceptable recovery time
- Retention requirement
- Restore-validation method
- Dependencies not protected by VM backup

Example lab targets:

- Daily recovery point
- 7–14 day retention
- Alternate restore validation
- No claim of cross-region failover

---

# 2. Create the Recovery Services Vault

1. Open **Azure Portal → Recovery Services vaults → Create**.
2. Configure:
   - Resource group: `rg-secure-ai-prod`
   - Name: `rsv-contoso-ai`
   - Region: Same region as the protected VM where required
3. Review and create.
4. Open the vault.

## Validation

- [ ] Vault deployed
- [ ] Correct subscription
- [ ] Correct region
- [ ] Azure RBAC model understood

---

# 3. Review Vault Security Settings

1. Open **Properties** or **Security settings**.
2. Review:
   - Soft delete
   - Enhanced soft delete
   - Multi-user authorization, if available and practical
   - Immutability options
   - Cross-subscription restore settings
3. Record the selected settings.
4. Consider adding a resource lock after configuration is stable.

> Do not enable irreversible controls casually in a disposable lab. Document the production recommendation when cleanup requirements differ.

---

# 4. Create the Backup Policy

1. Open **Vault → Backup policies → Add**.
2. Select Azure Virtual Machine.
3. Name: `bp-contoso-ai-daily`.
4. Configure:
   - Daily schedule
   - Appropriate local time
   - Daily retention of 7–14 days
   - Minimal instant-restore snapshot retention consistent with the lab
5. Save.

## Validation

Confirm schedule, time zone, retention, and policy name.

---

# 5. Enable Backup for the VM

1. Open **Vault → Backup**.
2. Workload location: Azure.
3. Workload type: Virtual machine.
4. Select `bp-contoso-ai-daily`.
5. Select `vm-ai-workload-test`.
6. Review disk inclusion.
7. Enable backup.

## Validation

- [ ] VM appears under Backup Items
- [ ] Policy assigned
- [ ] Backup status healthy or initial backup pending

---

# 6. Trigger an On-Demand Backup

1. Open the VM backup item.
2. Select **Backup now**.
3. Choose a retention date.
4. Start the job.
5. Monitor **Backup Jobs**.

## Validation

Confirm the job completes successfully and a recovery point appears.

---

# 7. Configure Backup Monitoring

1. Review **Backup Jobs** and **Alerts**.
2. Configure Azure Monitor alerting where practical for:
   - Backup failure
   - Restore failure
   - Backup protection stopped
3. Route notifications to the project administrator.
4. Document the alert configuration.

---

# 8. Perform an Alternate Restore Test

Preferred safe methods:

- Restore managed disks
- Create a new VM in `rg-contoso-ai-restore-test`
- Restore files, if supported and appropriate

Steps:

1. Open the recovery point.
2. Select **Restore VM** or **Restore disks**.
3. Choose an alternate target.
4. Use `rg-contoso-ai-restore-test`.
5. Avoid overwriting `vm-ai-workload-test`.
6. Start the restore.
7. Monitor the restore job.

---

# 9. Validate the Restored Workload

Validate one or more:

- Restored disk exists
- VM boots successfully
- Expected files are present
- Network settings are intentionally isolated
- No public IP was introduced
- Restored resource is not accidentally connected to production-style workflows

Document:

- Restore start and completion time
- Validation performed
- Any manual steps
- Cleanup plan

---

# 10. Document Recovery Boundaries

Record that Azure VM Backup does not automatically recover:

- Entra users and groups
- Conditional Access
- Azure Policy definitions/assignments
- Sentinel analytics logic stored only in the portal
- Azure AI resource configuration
- Private DNS and network topology
- Foundry project settings

Recovery methods for those controls:

- Git repository
- Exported KQL and playbooks
- Phase 9 Bicep
- Documented manual tenant procedures

---

# 11. Completion Checklist

## Vault

- [ ] Vault deployed
- [ ] Security settings reviewed
- [ ] Soft delete posture documented
- [ ] Lock considered

## Policy

- [ ] Daily policy created
- [ ] Retention configured
- [ ] Time zone confirmed
- [ ] Cost-conscious settings used

## Protection

- [ ] VM protected
- [ ] Disk selection reviewed
- [ ] On-demand backup completed
- [ ] Recovery point visible

## Restore

- [ ] Alternate restore initiated
- [ ] Restore job completed
- [ ] Restored disk/VM validated
- [ ] Production-style VM not overwritten
- [ ] Temporary resources identified for cleanup

## Documentation

- [ ] RPO/RTO assumptions recorded
- [ ] Recovery owner recorded
- [ ] PaaS/IaC recovery boundaries documented
- [ ] Screenshots captured

---

# 12. Troubleshooting

## VM does not appear for protection

Check region, subscription, existing vault protection, unsupported disk configuration, and permissions.

## Backup job remains pending

Check VM agent health, extension deployment, vault registration, platform status, and job details.

## Backup fails

Review the error code, disk state, VM agent, encryption compatibility, networking, and service-health notices.

## Restore cannot create a VM

Restore disks first, confirm target network/subnet availability, validate permissions, and review naming conflicts.

## Vault cannot be deleted

Check protected items, soft-deleted items, registered containers, private endpoints, resource locks, and immutability settings.

---

## Related Documentation

- [Phase 6 Portfolio Case Study](../06-business-continuity-recovery.md)
- [Phase 5 — Detection Engineering](../05-detection-engineering.md)
- [Phase 7 — Multi-Tenant Administration](../07-multi-tenant-administration.md)
- [Project Overview](../../README.md)

---

<div align="center">

**Runbook complete — a backup is not considered validated until a controlled restore has succeeded.**

</div>
