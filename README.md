# TryHackMe: Active Directory Basics Lab — Hands-On Documentation

### 🏅 Lab Verification Profile
* **Platform Host:** [TryHackMe](https://tryhackme.com/)
* **Target Module:** Windows and AD Fundamentals
* **Lab Environment:** `THM.local` (Windows Server Enterprise Domain Controller Simulation)
* **Verification Status:** Room Completed (100%)


<img src="https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/Screenshot%20(867).png" width="100%" alt="Active Directory Basics Completion Banner">

---

## 🎯 1. Executive Objective & Project Perspective
From an enterprise perspective, decentralized network architecture introduces massive operational overhead and significant security risks. Manually configuring local credentials, individual security properties, and workstation preferences across hundreds of endpoints is an unsustainable model for modern corporate environments.

This laboratory document outlines the practical implementation and administration of a centralized network environment. Utilizing Microsoft's **Active Directory Domain Services (AD DS)** on a dedicated **Domain Controller (DC)**, this repository documents the exact steps taken to establish a structured, scalable, and secure organizational environment.

---

## 🏗️ 2. Architectural Deployment (Windows Domains)

The initial phase involved reviewing a centralized domain infrastructure (`THM.local`). Moving away from isolated local computers allows all authentication processes to be forwarded back to a central repository—the Active Directory database (`ntds.dit`).

![Infrastructure Diagram](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/infras%20diagram.png)
> **IMAGE SOURCE:** Select the schematic image from the Task 2 overview displaying a workstation communicating with a central database host server.
> **CAPTION STRING:** *Fig. 1: Functional flow of centralized domain authentication vs. isolated local device account authentication.*

### Production Advantages Realized:
* **Centralized Identity Control:** User lifecycle events are managed from a single repository, making credentials universally accessible across authorized domain assets.
* **Uniform Security Baselines:** Global policies are push-deployed directly to network assets, eliminating configuration drift across business endpoints.

---

## 🏛️ 3. Object Classification & Directory Organization

Active Directory tracks assets as "objects" (Users, Machines, Security Groups). To maintain order, the domain infrastructure was partitioned using **Organizational Units (OUs)** to mirror the corporate organizational chart.

`[PLACEHOLDER: Insert Image 2 - OU Hierarchy Structure]`
> **IMAGE SOURCE:** Capture the Active Directory Users and Computers (ADUC) management console dashboard view displaying the THM folder and its departmental child folders.
> **CAPTION STRING:** *Fig. 2: Structural tree of departmental OUs utilized to apply targeted security configurations and manage structural lifecycles.*

### Core Administration Operations:
1. **Directory Cleanups (Simulated Decommissioning):** Audited the directory to remove old, inactive organizational units resulting from corporate restructuring. To execute this, **Advanced Features** were enabled in the View menu to uncheck "Protect object from accidental deletion" within the OU properties.
2. **Asset Segregation:** Established distinct baseline storage locations for devices by moving standard client hosts out of the generic `Computers` bin and sorting them into targeted **Workstations** and **Servers** containers.
3. **Privilege Auditing:** Documented default structural high-privilege groups to map administrative boundaries:
    * `Domain Admins`: Complete system access over all objects and DCs.
    * `Account Operators`: Permissions to create, update, or remove standard user identities.
    * `Server Operators`: Rights to manage domain hardware states and services.

---

## 🛠️ 4. Helpdesk Operations & Administrative Delegation

To avoid the dangerous security practice of granting full Domain Admin rights to standard support staff, **Role-Based Access Control (RBAC)** was enforced by delegating limited control over specific organizational paths.

### Least Privilege Enforcement
Control over the `Sales` user directory was safely delegated to a technician account (`Phillip`) specifically to manage credential issues. This provides helpdesk capabilities without introducing excessive lateral risk.

#### Helpdesk Administrative Automation (PowerShell)
Routine identity management tasks were processed efficiently using interactive command-line environments within the administrative workstation session:

`[PLACEHOLDER: Insert Image 3 - Interactive PowerShell Terminal session]`
> **IMAGE SOURCE:** Extract the terminal snapshot block showing the execution of the administrative identity scripts via the host shell.
> **CAPTION STRING:** *Fig. 3: Executing administrative identity automation scripts via PowerShell to enforce credential lifecycle management.*

```powershell
# Step 1: Securely reset a target user account password within the delegated OU space
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose

# Step 2: Enforce corporate policy by forcing a password alteration during the next user interactive logon session
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose

```

---

## 🛡️ 5. Baseline Security Policy & Group Policy (GPO) Deployment

Operational compliance and security restrictions were pushed globally across all endpoints via **Group Policy Objects (GPOs)** managed and distributed through the domain's network-wide `SYSVOL` share.

`[PLACEHOLDER: Insert Image 4 - Group Policy Management Editor]`

> **IMAGE SOURCE:** Capture the Policy Management window snapshot highlighting the configuration tree for Account Policies and password parameter alterations.
> **CAPTION STRING:** *Fig. 4: Hardening domain endpoints using the Group Policy Management Editor to configure account and preference baselines.*

### Implemented GPO Configurations:

* **Password Length Policy:** Enforced a baseline minimum length restriction of **10 characters** within the *Default Domain Policy* to eliminate weak user-defined credentials.
* **Session Inactivity Protection:** Configured a global endpoint policy to automatically trigger standard screen locks after **5 minutes** of continuous system inactivity, securing unattended machines.
* **Control Panel Restrictions:** Deployed a targeted custom GPO (`Restrict Control Panel Access`) linked explicitly to non-IT departments (`Management`, `Marketing`, `Sales`) to prevent unprivileged users from altering local operating system parameters.
* **Immediate Baseline Enforcement:** Leveraged the command `gpupdate /force` on endpoints to pull latest policies immediately, bypassing the default 2-hour update delay.

---

## 🔑 6. Network Authentication Mechanisms

To validate credentials across the domain without sending plain-text values across the network, two primary protocols were mapped:

### A. Kerberos Authentication (Default Enterprise Architecture)

`[PLACEHOLDER: Insert Image 5 - Kerberos Step Diagrams]`

> **IMAGE SOURCE:** Select the network interaction flowchart from the authentication module depicting the client, Key Distribution Center (KDC), and ticket validation events.
> **CAPTION STRING:** *Fig. 5: Multi-step authentication flow mapping Ticket Granting Tickets (TGT) and Ticket Granting Services (TGS).*

1. **Authentication Service:** Client requests a **Ticket Granting Ticket (TGT)** from the Key Distribution Center (KDC).
2. **Ticket Granting Service:** Client passes the encrypted TGT to the KDC to acquire a specific **Ticket Granting Service (TGS)** ticket for a requested resource.
3. **Application Session:** The TGS ticket is forwarded directly to the destination resource server to securely authenticate the session without revealing password material.

### B. NetNTLM (Legacy Challenge-Response Fallback)

Maintained strictly for backward compatibility, authentication relies on a standard challenge-response exchange:

1. The client attempts connection; the destination server responds with a random numeric **Challenge**.
2. The client encrypts the challenge using the local password **hash** and returns the calculation as a **Response**.
3. The server forwards this payload to the Domain Controller, which recalculates the challenge against the master database record to confirm access rights.

```

```
