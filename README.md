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
![High Privilege Administrator Configuration](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/admin.png)
* **Explanation:** This view demonstrates an account possessing global domain authority. It explicitly displays membership within high-privilege structural groups (`Domain Admins`), granting total configuration control across the entire domain infrastructure.

#### 2. Targeted Role-Based Access Control (The Technician Account)
![Technician Restricted Group Membership](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/PHILLIP.png)
* **Explanation:** This view confirms that the technician account (`Phillip`) remains restricted to standard `Domain Users` group privileges. He holds no global administrative groups, verifying that his helpdesk execution powers were securely bound strictly to the `Sales` OU container behind the scenes.

#### 3. Standard Non-Privileged Identity (Standard Corporate Employee)
![Standard Employee Account Configuration](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/thomas.png)
* **Explanation:** This view shows a baseline corporate user (`Thomas`). Like the technician, he is restricted to the standard `Domain Users` container group, ensuring zero structural configuration access over network resources.


#### Helpdesk Administrative Automation (PowerShell)
Routine identity management tasks were processed efficiently using interactive command-line environments within the administrative workstation session:

![PowerShell Target Prompt Input](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/search.png)
![Full PowerShell Automation Sequence](https://github.com/ThorisoM-hub/THM-active-directory-basics-lab/blob/main/images/powershell.png)
*Fig. 3b: The complete execution history in the shell. This window displays both commands running back-to-back, showing the masked password entry followed by the yellow `VERBOSE:` lines confirming the account flag was changed on the Domain Controller.*
*Fig. 3a: Initiating the `Set-ADAccountPassword` command for user 'sophie'. The console displays the secure `New Password:` interactive prompt, waiting for admin input.*
> **IMAGE SOURCE:** Extract the terminal snapshot block showing the execution of the administrative identity scripts via the host shell.
> **CAPTION STRING:** *Fig. 3: Executing administrative identity automation scripts via PowerShell to enforce credential lifecycle management.*


# Step 1: Securely reset a target user account password within the delegated OU space
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
* **What happens here:** This command targets the user account `sophie` and forces a password reset directly through the host shell interface.
* **The Security Mechanism:** Instead of typing the password out in plain text where a shoulder-surfer or local system log could capture it, the `(Read-Host -AsSecureString)` parameter masks the input on the screen with asterisks (`********`) and encrypts the password instantly inside the computer's memory.
* **The Output Indicator:** The yellow `VERBOSE:` lines confirm that the password alteration successfully reached and updated the object configuration on the Domain Controller.

# Step 2: Enforce corporate policy by forcing a password alteration during the next user interactive logon session
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
* **What happens here:** This command flags Sophie's account control attributes, forcing her to choose a brand-new password the very next time she attempts an interactive network logon.
* **The Security Principle:** This establishes a **Zero-Knowledge Architecture**. As a technician, I should never know a user's permanent password. By forcing her to overwrite my temporary password immediately, absolute accountability (non-repudiation) stays with Sophie. If her account is misused later, she cannot claim IT knew her password.
  
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

#### 4. Implementing Role-Based Access Control (RBAC)
* **The Solution:** By using the **Delegation of Control Wizard** in Active Directory, administrative permissions were securely scoped down to the absolute minimum required:
  * **Scope Limitation:** Restricted exclusively to the `Sales` OU container path. The technician account cannot view or modify objects inside the `IT`, `Management`, or `Marketing` OUs.
  * **Permission Limitation:** Restricted strictly to password management properties (specifically resetting passwords and forcing password changes at the next user logon session).

> 🛡️ **IAM / SOC Career Alignment:** Implementing targeted organizational delegation is a foundational pillar of modern IAM engineering. This exercise demonstrates a practical mastery of enforcing the **Principle of Least Privilege (PoLP)** at the structural level, a core requirement for mitigating insider threats and restricting lateral compromise vectors during an incident response scenario.

#### 4.1. Preventing Plain-Text Password Leaks
🔨 Step 1: The Secure Password Reset
PowerShell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
What it does: Resets the password for a user named sophie.

The Security Mechanism: Instead of typing her new password in plain text where anyone looking at the screen (or checking the command logs) could steal it, the (Read-Host -AsSecureString) part hides the typing. Windows instantly encrypts it in memory to keep it safe.

The -Verbose Flag: Forces PowerShell to show the step-by-step technical details of the reset in real time instead of processing it silently.

🔒 Step 2: Enforcing the Password Policy
PowerShell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
What it does: Flips a flag on Sophie's account so that the next time she logs in, Windows forces her to choose a new password.

The Security Principle: As an IT or security professional, I should never know a user's permanent password. Forcing an immediate change ensures that only Sophie knows her credentials. This creates clear accountability—if the account does something malicious later, the user cannot claim the helpdesk technician knew their password.

🎯 Why I Prioritized PowerShell Automation Over the GUI
I integrated this PowerShell module into the project to show how to scale identity tasks and handle credentials securely:

Handling Incidents at Scale: Clicking through menus works fine for one user, but it creates a massive bottleneck during a security incident. If a phishing attack compromises 50 accounts, manual GUI resets take too long. This script demonstrates the foundational logic needed to reset or lock down hundreds of compromised accounts programmatically in seconds.

Securing System Logs: Standard command history files (ConsoleHost_history.txt) automatically save clear-text strings. By using memory-encrypted parameters like (-AsSecureString), I ensure that sensitive user credentials never leak into local log files where attackers could harvest them
* **The Vulnerability: Hardcoding passwords directly into a command string (like -NewPassword "Password123") is a massive security risk. Windows automatically saves command line history into a local text file (ConsoleHost_history.txt). If you type a password in plain text, it gets saved to that file where anyone or any malicious script can read it.

* **The Fix: Pairing Read-Host with the -AsSecureString parameter masks the password on the screen while you type. Windows instantly encrypts the input directly in the computer's memory, ensuring the password never drops into local text logs.

####4.2.. Creating Absolute Accountability (Non-Repudiation)
* **Zero-Knowledge Rule: Forcing a password change at next logon ensures the administrator or helpdesk technician only knows a temporary password that expires immediately.

* **The Protection: Once the user logs in and sets their own private password, they are the only person who knows it. This protects me as the technician; if that account commits a malicious insider action later, the user cannot blame the helpdesk by saying, "The IT guy knew my password." It creates a clear audit trail and absolute accountability.
The Real-World Story of Sophie’s Password Reset
Scenario:
Sophie works in the Sales department. She comes to your IT Support desk or calls you because she forgot her password over the weekend and has locked herself out of her computer.

Step 1: The Helpdesk Encounter (Verifying Identity)
Before you touch a single command, you must verify she is actually Sophie (security protocol). Once confirmed, you open your administrative PowerShell window.

Step 2: Running Step 1 (The Temporary Reset)
You type the first command:

PowerShell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
What happens on your screen: The terminal prompts you: New Password:.

What you do: You type a temporary password, for example: Welcome@2026!. As you type, you only see asterisks (*********). You hit Enter.

The Confirmation: Because you typed -Verbose, PowerShell prints a yellow line confirming it successfully changed the password object on the Domain Controller. There is no need for a "confirm password" box like the GUI because the secure string handles the data safely in memory.

Step 3: Running Step 2 (Locking the Policy)
Right after, you run the second command:

PowerShell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
What you do: You hit Enter. The yellow text confirms the flag is set to $true.

The Hand-off: You look at Sophie and say: "Okay Sophie, I have reset your account. Your temporary password is Welcome@2026!. Go ahead and log into your computer."

Step 4: What Sophie Sees (The Magic Part)
Sophie goes back to her desk. She types her username (sophie) and the temporary password (Welcome@2026!) into the standard Windows login screen.

Instead of letting her see her desktop, Windows immediately pops up a message on her monitor:

⚠️ "The user's password must be changed before signing in."

Sophie clicks "OK". Windows then shows her two boxes:

New Password

Confirm Password

Sophie types in her private, permanent password (that only she knows) and hits Enter. Windows says "Your password has been changed," and she logs into her desktop safely.
🛡️ IAM / SOC Career Alignment: Security operations scale through automation. Mastering Active Directory management via PowerShell is an essential asset for IAM provisioning workflows and incident response isolation procedures (such as rapidly locking down or resetting compromised credentials en masse during a ransomware outbreak).




#### 1. Preventing Plain-Text Password Leaks

* **The Vulnerability:** Hardcoding raw passwords directly into a script string (like `-NewPassword "Password123"`) is a critical risk because Windows automatically saves command line histories into a plain text file (`ConsoleHost_history.txt`). Anyone or any malicious discovery script gaining access to the endpoint can easily harvest those credentials.
* **The Defensive Implementation:** Using masked input fields dynamically hides credentials from the display and completely bypasses clear-text storage within local asset logging boundaries.

#### 2. Creating Absolute Accountability

* **The Strategic Outcome:** Forcing password updates at first logon shields support personnel from false insider-threat accusations. It guarantees that the user is the sole custodian of their identity, preserving a flawless forensic audit trail.

> 🛡️ **IAM / SOC Career Alignment:** Security operations scale through automation. Mastering Active Directory management via PowerShell is an essential asset for IAM provisioning workflows and incident response isolation procedures (such as rapidly locking down or resetting compromised credentials en masse during a ransomware outbreak).

---

### 💡 Why this works perfectly for your repo:

It doesn't use over-complicated, high-level jargon. It explains **exactly what happens on the screen** (the asterisks, the yellow text, the next login prompt) so a recruiter reading it immediately knows *you* actually performed the lab, understood the mechanics, and knew the exact defensive reasoning behind the commands.
