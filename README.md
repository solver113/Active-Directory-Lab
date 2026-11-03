# Enterprise Active Directory & Security Hardening Lab

## Executive Summary
This project documents an enterprise-grade Active Directory infrastructure built under the **Solver Cybertech** engineering lab framework. It covers the full lifecycle from Windows Server 2025 Domain Controller promotion and client domain joining, to advanced Identity & Access Management (IAM), Role-Based Access Control (RBAC), Group Policy Object (GPO) security hardening, and PowerShell detection/audit logging telemetry.

---

## Technical Overview
* **Domain Name:** `solver.local`
* **Domain Controller OS:** Windows Server 2025 Standard Evaluation
* **Domain Controller IP:** `192.168.245.162` (Static)
* **Client OS:** Windows Workstation (Domain-Joined)
* **Identity Schema:** Custom OU (`solver_users`), Standard Users (`jane`, `goodluck`), & Security Groups (`rdp access`)

---

## Deployment & Defensive Hardening Phases

### Phase 1: Core DC Infrastructure & Domain Join
* Promoted host to Primary Domain Controller (`solver.local`) with localized DNS authority (`127.0.0.1`).
* Configured client DNS to target the DC (`192.168.245.162`) and completed the workstation domain join.

![Static IP Configuration](01-static-ip-config.png)
![AD DS Role Selection](02-ad-ds-role-selection.png)
![Forest Configuration](03-forest-config.png)
![Domain Login Verification](04-post-promotion-login.png)
![Active Directory Structure](05-ad-structure-verification.png)
![Workstation DNS Configuration](07-workstation-dns-config.png)
![Domain Join Success](08-domain-join-success.png)

---

### Phase 2: IAM Structuring & RBAC Security Groups
1. Established the `solver_users` Organizational Unit (OU).
2. Provisioned standard domain user accounts (`jane`, `goodluck`) and administrative accounts (`solver admin`).
3. Created the **`rdp access`** Global Security Group and assigned `jane` as a member to enforce Role-Based Access Control (RBAC).

![Active Directory Users and OUs](06-ad-user-creation.png)
![RDP Security Group Configuration](10-rdp-security-group.png)

---

### Phase 3: GPO Hardening — Least-Privilege RDP Enforcement
To restrict lateral movement, generic Remote Desktop permissions were revoked enterprise-wide via Group Policy.

1. **GPO Creation:** Created and linked `Restrict_RDP_Policy` at the domain root level.
2. **User Rights Assignment:** Configured *Allow log on through Remote Desktop Services* to grant explicit rights **ONLY** to `SOLVER\Domain Admins` and `SOLVER\rdp access`.
3. **Validation & Negative Testing:** 
   * **Authorized User (`jane`):** RDP authentication succeeded via `SOLVER\rdp access` membership.
   * **Unauthorized User (`goodluck`):** RDP authentication was explicitly denied by security policy.

![GPO Domain Link](11-gpo-linked-ou.png)
![User Rights Assignment Policy](12-user-rights-assignment.png)
![Authorized RDP Session Success](13-rdp-access-success.png)
![Unauthorized RDP Access Denied](14-rdp-access-denied.png)

---

### Phase 4: GPO Hardening — PowerShell Execution Restriction
Unrestricted script execution presents a severe risk for fileless malware staging. Execution controls were enforced enterprise-wide via GPO.

1. **Baseline State:** Demonstrated successful execution of an unsigned PowerShell script (`test.ps1`) prior to policy enforcement.
2. **GPO Hardening:** Enforced `Restrict_PowerShell_Execution` via Administrative Templates, requiring signed scripts domain-wide.
3. **Validation & A/B Testing:** Attempted execution of `test.ps1` post-enforcement, resulting in an immediate system execution policy block. Verified GPO precedence via `Get-ExecutionPolicy -List`.

![Unrestricted PowerShell Execution (Before)](15-powershell-execution-before.png)
![PowerShell Execution Policy GPO](16-gpo-powershell-policy.png)
![PowerShell GPO Domain Link](17-gpo-powershell-linked.png)
![PowerShell Execution Denied (After)](18-powershell-execution-denied.png)
![PowerShell Execution Policy Scope](19-powershell-policy-list.png)

---

### Phase 5: Audit & Detection Telemetry — PowerShell Script Block Logging
Restricting execution stops basic attacks, but enterprise SecOps requires audit trails to detect obfuscated commands and fileless threats.

1. **GPO Logging Baseline:** Created and linked `Enable_PowerShell_Logging` at the domain level.
   * **Module Logging:** Enabled for all modules (`*`).
   * **Script Block Logging:** Enabled to capture start/stop events and full execution payloads (Event ID 4104).
2. **Telemetry Validation:** Executed a test script block on a target workstation and inspected Windows Event Viewer (`Microsoft-Windows-PowerShell/Operational`).
3. **De-obfuscation Verification:** Confirmed that **Event ID 4104** recorded the exact script payload (`solver cybertech log verification test`) inside the log details.

![PowerShell Logging GPO Configured](20-gpo-logging-configured.png)
![PowerShell Logging GPO Linked](21-gpo-logging-linked.png)
![Event Viewer Event ID 4104 Logs](22-powershell-event-viewer-logs.png)
![Script Block Telemetry Capture](23-script-block-deobfuscation.png)

---

## Key Defensive Engineering Takeaways
* **Least Privilege Access:** Restricting RDP via User Rights Assignment neutralizes lateral movement attempts by unauthorized standard domain users.
* **Execution Prevention:** Centralized script execution controls prevent unapproved script execution and fileless malware staging.
* **Security Telemetry & Auditing:** Centralized Script Block Logging (Event ID 4104) ensures full visibility into PowerShell execution, capturing de-obfuscated script payloads for SIEM analysis and incident investigation.
* **Proof-of-Concept Rigor:** Combining positive access checks with explicit negative security testing and event audit verification proves security baselines actively enforce policy.

---

## Author
**Peter Okonkwo**  
Lead Engineer @ Solver Cybertech  
* GitHub: [github.com/solver113](https://github.com/solver113)  
* LinkedIn: [peter-okonkwo-3b6610230](https://www.linkedin.com/in/peter-okonkwo-3b6610230)
