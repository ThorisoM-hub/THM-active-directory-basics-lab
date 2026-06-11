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

![OU Hierarchy Structure](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/ou%20hierachy.png)
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



## 🔐 4.## 🔐 4. Helpdesk Operations, Delegation of Control & Role-Based Access Control (RBAC)

To avoid the dangerous security practice of granting full Domain Admin rights to standard support staff, Role-Based Access Control (RBAC) was enforced by delegating limited control over specific organizational paths.

### Least Privilege Enforcement

Control over the `Sales` user directory was safely delegated to a technician account (`Phillip`) specifically to manage credential issues. This provides helpdesk capabilities without introducing excessive lateral risk.

| User Account | What their "Member Of" tab shows | What their actual power is |
| :--- | :--- | :--- |
| **`Administrator`** | `Domain Admins`, `Enterprise Admins`, `Schema Admins` | **Total Control:** Can modify the entire network, access every server, change any security policy, and delete any directory. |
| **`Phillip`** *(The Technician)* | **`Domain Users`** | **Targeted Control (RBAC):** He looks like a regular user, but because of the Delegation Wizard, he has an explicit rule on the Sales OU that lets him reset passwords for that team only. |
| **`Thomas / Claire`** *(Standard Employees)* | **`Domain Users`** | **Zero Administrative Power:** Regular staff who can only log into their assigned workstations and do their daily work. |

---

### 📊 Structural Verification & Directory Auditing

#### 1. Over-Privileged Account Verification (The Administrator Account)
![High Privilege Administrator Configuration](images/admin_privileges.png)
* **Explanation:** This view demonstrates an account possessing global domain authority. It explicitly displays membership within high-privilege structural groups (`Domain Admins`), granting total configuration control across the entire domain infrastructure.

#### 2. Targeted Role-Based Access Control (The Technician Account)
![Technician Restricted Group Membership](images/delegation_control.png)
* **Explanation:** This view confirms that the technician account (`Phillip`) remains restricted to standard `Domain Users` group privileges. He holds no global administrative groups, verifying that his helpdesk execution powers were securely bound strictly to the `Sales` OU container behind the scenes.

#### 3. Standard Non-Privileged Identity (Standard Corporate Employee)
![Standard Employee Account Configuration](images/standard_user_privileges.png)
* **Explanation:** This view shows a baseline corporate user (`Thomas`). Like the technician, he is restricted to the standard `Domain Users` container group, ensuring zero structural configuration access over network resources.


#### Helpdesk Administrative Automation (PowerShell)
Routine identity management tasks were processed efficiently using interactive command-line environments within the administrative workstation session:

`[PLACEHOLDER: Insert Image 3 - Interactive PowerShell Terminal session]`
> **IMAGE SOURCE:** Extract the terminal snapshot block showing the execution of the administrative identity scripts via the host shell.
> **CAPTION STRING:** *Fig. 3: Executing administrative identity automation scripts via PowerShell to enforce credential lifecycle management.*


# Step 1: Securely reset a target user account password within the delegated OU space
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose

# Step 2: Enforce corporate policy by forcing a password alteration during the next user interactive logon session
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose

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
### 📝 Key Operational Notes & Security Takeaways

To better understand the real-world impact of these directory operations, the following core concepts were analyzed during the lab:

#### 1. Directory Cleanups (Simulated Decommissioning)
* **The Concept:** Think of this as decommissioning old, stale assets on an enterprise network. When a department closes down or merges, maintaining a clean security posture requires removing its old Organizational Unit (OU).
* **The Safety Mechanisms:** Windows Server protects OUs by default to prevent a rogue administrator or an accidental click from wiping out an entire department (along with all its users and group policies). Decommissioning an OU requires explicitly enabling **Advanced Features** in the *View* menu and unchecking **"Protect object from accidental deletion"** within the object properties. Documenting this step proves an understanding of safe enterprise change-management workflows.

#### 2. Asset Segregation (Hardening the Default State)
* **The Concept:** When a new workstation or server joins a Windows domain, Active Directory automatically places it into a generic, flat container called **`Computers`**. 
* **The Security Risk:** Leaving assets in the default container is a dangerous security practice because you cannot link targeted **Group Policy Objects (GPOs)** to generic default containers. 
* **The Solution:** By moving machines out of the generic pile and sorting them into dedicated, custom OUs (e.g., separating standard client laptops from critical database host servers), strict security boundaries can be enforced. For instance, restrictive firewall configurations can be applied exclusively to servers while maintaining standard access policies for everyday employee workstations.

#### 3. Privilege Auditing (Mapping the Keys to the Kingdom)
Analyzing the structural administrative boundaries of default high-privilege built-in groups ensures compliance with the **Principle of Least Privilege (PoLP)**:
* 👑 **Domain Admins:** Possess complete administrative control over all objects, workstations, member servers, and Domain Controllers across the entire infrastructure. If a threat actor compromises a Domain Admin credential, it results in complete network takeover.
* 👥 **Account Operators:** Restricted to identity management workflows. They can create, update, or decommission standard user accounts and security groups, but lack permissions to modify core system configurations or compromise administrative credentials.
* ⚙️ **Server Operators:** Tasked with maintaining physical or virtual hardware states. They can log onto domain servers interactively, restart system services, configure backup routines, and format storage volumes, but do not manage identity lifecycles.

> 🛡️ **IAM / SOC Career Alignment:** Documenting this phase demonstrates a practical grip on structural identity placement and access boundaries—key competencies required to identify unauthorized privilege escalations during active security monitoring or access reviews.
To better understand the real-world impact of Active Directory delegation, the following architectural concepts were analyzed during the lab:

#### 1. The Danger of Over-Privileged Service Accounts
* **The Real-World Issue:** In legacy or poorly managed corporate environments, helpdesk staff or IT support technicians are frequently added directly to the `Domain Admins` group simply because they need to perform basic identity management tasks like resetting passwords or unlocking accounts. 
* **The Security Risk:** If a technician's account is compromised via phishing or credential harvesting, the attacker instantly inherits full, domain-wide administrative rights. This allows immediate lateral movement and total infrastructure takeover.

#### 2. Implementing Role-Based Access Control (RBAC)
* **The Solution:** By using the **Delegation of Control Wizard** in Active Directory, administrative permissions were securely scoped down to the absolute minimum required:
  * **Scope Limitation:** Restricted exclusively to the `Sales` OU container path. The technician account cannot view or modify objects inside the `IT`, `Management`, or `Marketing` OUs.
  * **Permission Limitation:** Restricted strictly to password management properties (specifically resetting passwords and forcing password changes at the next user logon session).

> 🛡️ **IAM / SOC Career Alignment:** Implementing targeted organizational delegation is a foundational pillar of modern IAM engineering. This exercise demonstrates a practical mastery of enforcing the **Principle of Least Privilege (PoLP)** at the structural level, a core requirement for mitigating insider threats and restricting lateral compromise vectors during an incident response scenario.
