# Enterprise Active Directory, Defensive Hardening & Disaster Recovery Lab

## Executive Summary
This repository documents an enterprise-grade Active Directory infrastructure built under the **Solver Cybertech** engineering lab framework. It demonstrates end-to-end expertise spanning Windows Server 2025 Domain Controller deployment, client domain onboarding, Identity & Access Management (IAM), Role-Based Access Control (RBAC), Group Policy Object (GPO) security hardening, PowerShell detection telemetry (Event ID 4104), and Volume Shadow Copy Service (VSS) disaster recovery baselines.

---

## Technical Infrastructure Overview
* **Domain Name:** `solver.local`
* **Domain Controller Host:** Windows Server 2025 Standard Evaluation
* **Domain Controller IP:** `192.168.245.162` (Static)
* **Client Host:** Windows Workstation (Domain-Joined)
* **Identity Schema:** Custom OU (`solver_users`), Standard Accounts (`jane`, `goodluck`), & Security Groups (`rdp access`)
* **Network Storage:** Centralized SMB File Share (`\\SOLVER\CompanyShare`)

---

## Deployment & Defensive Hardening Phases

### Phase 1: Core DC Infrastructure & Domain Join
* Promoted primary Domain Controller (`solver.local`) with localized DNS authority (`127.0.0.1`).
* Configured client workstation DNS (`192.168.245.162`) and executed domain join.

![Static IP Configuration](01-static-ip-config.png)
![AD DS Role Selection](02-ad-ds-role-selection.png)
![Forest Configuration](03-forest-config.png)
![Domain Login Verification](04-post-promotion-login.png)
![Active Directory Structure](05-ad-structure-verification.png)
![Workstation DNS Configuration](07-workstation-dns-config.png)
![Domain Join Success](08-domain-join-success.png)

---

### Phase 2: IAM Structuring & RBAC Security Groups
1. Provisioned `solver_users` Organizational Unit (OU).
2. Created user accounts (`jane`, `goodluck`) and administrative identities (`solver admin`).
3. Created **`rdp access`** Global Security Group to enforce Role-Based Access Control (RBAC).

![Active Directory Users and OUs](06-ad-user-creation.png)
![RDP Security Group Configuration](10-rdp-security-group.png)

---

### Phase 3: GPO Hardening — Least-Privilege RDP Enforcement
To neutralize lateral movement risks, generic Remote Desktop privileges were revoked domain-wide.

1. **GPO Deployment:** Created and linked `Restrict_RDP_Policy` at the domain root.
2. **User Rights Assignment:** Restricted *Allow log on through Remote Desktop Services* exclusively to `SOLVER\Domain Admins` and `SOLVER\rdp access`.
3. **A/B Access Verification:** 
   * **Authorized User (`jane`):** Successful RDP authentication via `rdp access` group membership.
   * **Unauthorized User (`goodluck`):** Access explicitly denied by domain policy.

![GPO Domain Link](11-gpo-linked-ou.png)
![User Rights Assignment Policy](12-user-rights-assignment.png)
![Authorized RDP Session Success](13-rdp-access-success.png)
![Unauthorized RDP Access Denied](14-rdp-access-denied.png)

---

### Phase 4: GPO Hardening — PowerShell Execution Restriction
Enforced centralized execution controls to prevent unauthorized script execution and fileless malware staging.

1. **Unrestricted Baseline:** Confirmed baseline execution of unsigned scripts (`test.ps1`).
2. **GPO Hardening:** Applied `Restrict_PowerShell_Execution` GPO, enforcing `AllSigned` restrictions domain-wide.
3. **Verification:** Validated that unsigned script execution was immediately blocked by policy and confirmed precedence via `Get-ExecutionPolicy -List`.

![Unrestricted PowerShell Execution (Before)](15-powershell-execution-before.png)
![PowerShell Execution Policy GPO](16-gpo-powershell-policy.png)
![PowerShell GPO Domain Link](17-gpo-powershell-linked.png)
![PowerShell Execution Denied (After)](18-powershell-execution-denied.png)
![PowerShell Execution Policy Scope](19-powershell-policy-list.png)

---

### Phase 5: Audit & Detection Telemetry — PowerShell Script Block Logging
Implemented centralized log auditing to maintain full visibility into fileless execution payloads and de-obfuscated script code.

1. **GPO Telemetry Baseline:** Created and linked `Enable_PowerShell_Logging` at the domain level.
   * **Module Logging:** Enabled enterprise-wide (`*`).
   * **Script Block Logging:** Enabled execution logging (Event ID 4104).
2. **Telemetry Validation:** Executed test payloads on a domain workstation and verified event ingestion in Windows Event Viewer (`Microsoft-Windows-PowerShell/Operational`).
3. **Payload Inspection:** Confirmed **Event ID 4104** captured the exact de-obfuscated script block text (`solver cybertech log verification test`).

![PowerShell Logging GPO Configured](20-gpo-logging-configured.png)
![PowerShell Logging GPO Linked](21-gpo-logging-linked.png)
![Event Viewer Event ID 4104 Logs](22-powershell-event-viewer-logs.png)
![Script Block Telemetry Capture](23-script-block-deobfuscation.png)

---

### Phase 6: Storage Resilience & Volume Shadow Copy (VSS) Disaster Recovery
To protect critical enterprise data against accidental deletion, ransomware impact, or user error, Volume Shadow Copy Service (VSS) and network discovery controls were deployed across SMB file shares.

1. **SMB Share Creation:** Configured centralized network share `\\SOLVER\CompanyShare` with domain access permissions.
2. **VSS Snapshot Baseline:** Enabled Volume Shadow Copies on the `C:` volume hosting `CompanyShare` and created automated/manual snapshot restore points.
3. **GPO Topology Control:** Enforced `Enable_Network_Discovery_VSS` GPO (Link-Layer Topology Discovery) so domain workstations map and discover hosts seamlessly.
4. **Disaster Recovery Testing:** Simulated accidental deletion of `Solver_Cybertech_Confidential.txt` on a domain client and successfully restored the file intact using **Previous Versions** snapshot recovery.

![Volume Shadow Copies Enabled](24-vss-enabled-server.png)
![Network Discovery GPO Configured](25-gpo-network-discovery.png)
![File Deletion Baseline](26-file-deletion-baseline.png)
![Shadow Copy File Restored](27-shadow-copy-restored.png)

---

## Key Defensive Engineering & Systems Engineering Takeaways
* **Least Privilege Access:** Restricting RDP via User Rights Assignment neutralizes lateral movement by standard users.
* **Execution Prevention & Telemetry:** Centralized PowerShell execution controls paired with Event ID 4104 Script Block Logging ensure full visibility into malicious script execution for SIEM ingestion.
* **Storage Resilience & RPO Minimization:** Deploying Volume Shadow Copies guarantees near-instant file recovery from accidental user deletion without requiring full system backup restores.
* **A/B Testing Rigor:** Combining positive access tests with explicit negative security checks and recovery tests proves security controls function properly under enterprise conditions.

---

## Author
**Peter Okonkwo**  
Lead Engineer @ Solver Cybertech  
* GitHub: [github.com/solver113](https://github.com/solver113)  
* LinkedIn: [peter-okonkwo-3b6610230](https://www.linkedin.com/in/peter-okonkwo-3b6610230)
