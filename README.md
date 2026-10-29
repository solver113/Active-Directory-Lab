
# Active Directory Domain Controller Deployment

## Executive Summary
This project documents the installation, configuration, and promotion of a Windows Server 2025 Domain Controller within an isolated enterprise virtual lab environment. Built under the **Solver Cybertech** engineering lab, this deployment serves as the foundational Identity and Access Management (IAM) infrastructure for subsequent Active Directory security testing, Group Policy Object (GPO) deployment, and defensive SecOps monitoring.

---

## Technical Overview
* **OS Version:** Windows Server 2025 Standard Evaluation
* **Domain Name:** `solver.local`
* **Server Role:** Primary Domain Controller (PDC) & DNS Server
* **Networking Mode:** Virtual NAT Network
* **IPv4 Address:** `192.168.245.162` (Static)
* **Default Gateway:** `192.168.245.2`
* **Preferred DNS:** `127.0.0.1` (Loopback Authority)

---

## Deployment & Execution Steps

### Phase 1: Prerequisite Staging & Static IP Binding
Before promoting a server to a Domain Controller, a persistent static IP configuration is required to prevent DNS resolution failures across domain-joined hosts.

* Configured static TCP/IPv4 parameters matching the NAT subnet range.
* Set the preferred DNS server to `127.0.0.1` so the host targets its local Active Directory DNS service once installed.

![Static IP Configuration](01-static-ip-config.png)

---

### Phase 2: Role Installation & Forest Promotion
1. Initialized the **Add Roles and Features Wizard** in Server Manager and selected **Active Directory Domain Services (AD DS)** along with integrated DNS features.
2. Selected **"Add a new forest"** and established the Root Domain Name as `solver.local`.
3. Configured the Directory Services Restore Mode (DSRM) password and proceeded through prerequisite checks.

![AD DS Role Selection](02-ad-ds-role-selection.png)
![Forest Configuration](03-forest-config.png)

---

### Phase 3: Post-Promotion Verification & Management
Following server reboot and promotion completion:
* Verified domain authority via administrator authentication (`SOLVER\Administrator`).
* Confirmed domain schema, built-in containers, and default OUs using the **Active Directory Administrative Center**.

![Domain Login Verification](04-post-promotion-login.png)
![Active Directory Administrative Center](05-ad-structure-verification.png)

---

## Key Takeaways
* **DNS Dependency:** Active Directory relies entirely on correct DNS architecture. Pointing the network interface to loopback (`127.0.0.1`) ensures localized authority for all future domain queries.
* **Isolated Forest Staging:** Warnings regarding public DNS delegation can be safely bypassed in localized lab environments as internal root authority is generated locally.

---

## Author
**Peter Okonkwo**  
Lead Engineer @ Solver Cybertech  
* GitHub: [github.com/solver113](https://github.com/solver113)  
* LinkedIn: [peter-okonkwo-3b6610230](https://www.linkedin.com/in/peter-okonkwo-3b6610230)
