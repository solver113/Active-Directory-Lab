# Enterprise Active Directory, Security Hardening & Remote Management Lab

## Executive Summary
This repository documents a production-grade, enterprise Active Directory infrastructure engineered under the **Solver Cybertech** technical brand. Built on **Windows Server 2025 Standard**, this project provides full architectural proof-of-work across Domain Services, Identity & Access Management (IAM), Role-Based Access Control (RBAC), Group Policy Object (GPO) security baselines, PowerShell detection telemetry (Event ID 4104), Volume Shadow Copy Service (VSS) data protection, and enterprise WinRM remote management automation.

---

## Technical Architecture & Identity Schema
* **Domain Name:** `solver.local`
* **Domain Controller Host:** Windows Server 2025 Standard Evaluation
* **Domain Controller IP:** `192.168.245.162` (Static)
* **Client Endpoints:** Windows Workstation (Domain-Joined)
* **Directory Structure:** Custom OU (`solver_users`), Standard Users (`jane`, `goodluck`), Administrative Users (`solver admin`), Security Groups (`rdp access`)
* **Storage Resilience:** Centralized SMB File Share (`\\SOLVER\CompanyShare`) with VSS integration

---

## Architectural Implementation & Engineering Phases

### Phase 1: Core DC Infrastructure & Client Onboarding
* Provisioned Primary Domain Controller (`solver.local`) with static network parameters and localized DNS authority (`127.0.0.1`).
* Configured client DNS binding (`192.168.245.162`) and executed seamless workstation domain joining.

![Static IP Configuration](01-static-ip-config.png)
![AD DS Role Selection](02-ad-ds-role-selection.png)
![Forest Configuration](03-forest-config.png)
![Domain Login Verification](04-post-promotion-login.png)
![Active Directory Structure](05-ad-structure-verification.png)
![Workstation DNS Configuration](07-workstation-dns-config.png)
![Domain Join Success](08-domain-join-success.png)

---

### Phase 2: Identity & Access Management (IAM) and RBAC Structure
1. Structured Organizational Unit layout (`solver_users`) to segregate accounts from default containers.
2. Provisioned standard domain user accounts and dedicated administrative accounts.
3. Created **`rdp access`** Global Security Group to enforce Role-Based Access Control (RBAC).

![Active Directory Users and OUs](06-ad-user-creation.png)
![RDP Security Group Configuration](10-rdp-security-group.png)

---

### Phase 3: GPO Security Hardening — Least-Privilege RDP Enforcement
Mitigated lateral movement risks by revoking broad Remote Desktop privileges across domain endpoints.

1. **Policy Deployment:** Enforced `Restrict_RDP_Policy` GPO at the domain root level.
2. **User Rights Assignment:** Restricted *Allow log on through Remote Desktop Services* strictly to `SOLVER\Domain Admins` and `SOLVER\rdp access`.
3. **Negative Access Controls Validation:** 
   * **Authorized Identity (`jane`):** RDP authentication succeeded via explicit `rdp access` group membership.
   * **Unauthorized Identity (`goodluck`):** Access explicitly denied by security policy baseline.

![GPO Domain Link](11-gpo-linked-ou.png)
![User Rights Assignment Policy](12-user-rights-assignment.png)
![Authorized RDP Session Success](13-rdp-access-success.png)
![Unauthorized RDP Access Denied](14-rdp-access-denied.png)

---

### Phase 4: GPO Security Hardening — PowerShell Execution Controls
Established enterprise script execution baselines to prevent unauthorized code execution and fileless malware staging.

1. **Baseline State:** Verified baseline execution of unsigned scripts (`test.ps1`).
2. **Execution Hardening:** Deployed `Restrict_PowerShell_Execution` GPO, enforcing `AllSigned` restrictions enterprise-wide.
3. **A/B Policy Verification:** Confirmed immediate system-level execution blocks for unsigned code and validated policy precedence via `Get-ExecutionPolicy -List`.

![Unrestricted PowerShell Execution (Before)](15-powershell-execution-before.png)
![PowerShell Execution Policy GPO](16-gpo-powershell-policy.png)
![PowerShell GPO Domain Link](17-gpo-powershell-linked.png)
![PowerShell Execution Denied (After)](18-powershell-execution-denied.png)
![PowerShell Execution Policy Scope](19-powershell-policy-list.png)

---

### Phase 5: Security Auditing & Telemetry — Script Block Logging
Configured log auditing to maintain continuous visibility into obfuscated command execution and fileless threat activity.

1. **GPO Telemetry Baseline:** Applied `Enable_PowerShell_Logging` GPO at the domain level.
   * **Module Logging:** Enabled enterprise-wide (`*`).
   * **Script Block Logging:** Enabled execution logging (Event ID 4104).
2. **Telemetry Inspection:** Executed test commands on target endpoints and verified ingestion in Windows Event Viewer (`Microsoft-Windows-PowerShell/Operational`).
3. **De-obfuscation Verification:** Confirmed that **Event ID 4104** captured de-obfuscated script block payloads (`solver cybertech log verification test`) for SIEM ingestion.

![PowerShell Logging GPO Configured](20-gpo-logging-configured.png)
![PowerShell Logging GPO Linked](21-gpo-logging-linked.png)
![Event Viewer Event ID 4104 Logs](22-powershell-event-viewer-logs.png)
![Script Block Telemetry Capture](23-script-block-deobfuscation.png)

---

### Phase 6: Storage Resilience & Volume Shadow Copy (VSS) Disaster Recovery
Protected SMB file share assets against data loss and user error using point-in-time snapshot baselines.

1. **Storage Deployment:** Established `\\SOLVER\CompanyShare` with domain-level security permissions.
2. **VSS Snapshot Integration:** Enabled Volume Shadow Copies on host volumes (`C:`) to generate automated recovery points.
3. **Network Discovery GPO:** Deployed `Enable_Network_Discovery_VSS` GPO (Link-Layer Topology Discovery) to ensure seamless endpoint discovery.
4. **Disaster Recovery Validation:** Simulated accidental file deletion of critical assets (`Solver_Cybertech_Confidential.txt`) and completed point-in-time snapshot recovery via **Previous Versions**.

![Volume Shadow Copies Enabled](24-vss-enabled-server.png)
![Network Discovery GPO Configured](25-gpo-network-discovery.png)
![File Deletion Baseline](26-file-deletion-baseline.png)
![Shadow Copy File Restored](27-shadow-copy-restored.png)

---

### Phase 7: Enterprise Remote Management — WinRM & PowerShell Remoting GPO
Automated remote endpoint administration across domain hosts using secure, centralized Windows Remote Management (WinRM).

1. **Service Automation:** Configured `Enable_WinRM_Management` GPO to enforce **Windows Remote Management (WS-Management)** service startup as **Automatic**.
2. **Firewall Rule Enforcement:** Pushed GPO firewall rules enabling **Windows Remote Management (HTTP-In)** over TCP port 5985 domain-wide.
3. **WinRM Listener Configuration:** Enforced listener binding policies across all network adapters.
4. **Remoting Verification:** Established an interactive remote PowerShell session (`Enter-PSSession`) from the Domain Controller to target workstation endpoints to execute administrative tasks remotely.

![WinRM Service GPO Configured](28-gpo-winrm-service-configured.png)
![WinRM Inbound Firewall Policy](29-gpo-winrm-firewall-configured.png)
![WinRM GPO Domain Linked](30-gpo-winrm-linked.png)
![PowerShell Remoting Interactive Session](31-powershell-remoting-success.png)

---

## Core Technical Competencies Demonstrated
* **Directory & Identity Management:** Active Directory Domain Services (AD DS), DNS Server management, Organizational Units (OU), User Lifecycle Provisioning, Role-Based Access Control (RBAC).
* **Defensive Engineering & Group Policy:** GPO enforcement, User Rights Assignment, Least-Privilege access design, PowerShell execution controls (`AllSigned`).
* **Security Operations & Detection Telemetry:** Centralized Script Block Logging, Module Logging, Event ID 4104 audit verification, de-obfuscation inspection for SIEM readiness.
* **Storage Engineering & Disaster Recovery:** Volume Shadow Copy Service (VSS) configuration, SMB network sharing, Point-in-Time snapshot recovery (RPO minimization).
* **Remote Systems Management:** Windows Remote Management (WinRM), PowerShell Remoting (`Enter-PSSession`), centralized GPO Firewall rule management.
* **Engineering Methodology:** Rigorous A/B baseline security testing, positive access verification, and negative security testing.

---

## Author
**Peter Okonkwo**  
Lead Engineer @ Solver Cybertech  
* GitHub: [github.com/solver113](https://github.com/solver113)  
* LinkedIn: [peter-okonkwo-3b6610230](https://www.linkedin.com/in/peter-okonkwo-3b6610230)
