# Group Policy Object (GPO) Projects

This document details the core Group Policies implemented in the `myITlab.local` environment. These policies were designed following principle of least privilege, Microsoft Security Baselines, and organizational requirements.

---

## 1. Default Domain Policy (Password & Account Lockout)
**Goal:** Establish a baseline security standard for all user accounts across the domain to protect against brute-force attacks.
*   **Scope:** Linked to the root domain (`myITlab.local`).
*   **Key Configurations:**
    *   Enforced strong password complexity requirements.
    *   Configured Account Lockout Policy (e.g., locking accounts after a set number of failed attempts).
*   **Why it matters:** Ensures that every single user, regardless of their department OU, is held to a minimum cryptographic standard.

---

## 2. Advanced Audit Policy (Security Monitoring)
**Goal:** Increase visibility into critical infrastructure changes and privileged account usage.
*   **Scope:** Linked to the `Servers` and `DomainControllers` OUs.
*   **Key Configurations:**
    *   Enabled auditing for "Security Group Management" (to track when users are added/removed from admin groups).
    *   Enabled "Logon/Logoff" subcategories to track privileged access.
*   **Verification:** Validated by capturing Event ID 4728 on DC01 when adding a user to `GRP-Servers-Admin`. *(See [Auditing Evidence](Auditing-Evidence.md) for screenshots).*

---

## 3. Windows 10 Security Baseline & PowerShell Logging
**Goal:** Harden endpoint workstations and provide forensic visibility into script execution.
*   **Scope:** Linked to the `Workstations/Windows` OU.
*   **Key Configurations:**
    *   Applied settings based on Microsoft Security Baselines.
    *   Enabled PowerShell Module Logging and Script Block Logging.
    *   Disabled legacy protocols like SMBv1 and hardened NTLM/LDAP.
*   **Why it matters:** PowerShell logging is a critical defense mechanism against "living off the land" (LotL) cyber attacks.

---

## 4. Linux Access Control
**Goal:** Centralize authentication and restrict local logon access for Linux endpoints using Active Directory.
*   **Scope:** Linked to the `Workstations/Linux` OU.
*   **Key Configurations:**
    *   Granted "Allow log on locally" rights exclusively to the `GG_LinuxAdmins` group.
    *   Enforced via SSSD (`ad_gpo_access_control = enforcing`) on the Fedora client.
*   **Why it matters:** Demonstrates cross-platform enterprise management, ensuring standard Windows domain users cannot access Linux infrastructure.

---

## 5. Service Account Hardening
**Goal:** Prevent service accounts from being used interactively by threat actors.
*   **Scope:** Linked to the `Service Accounts` OU.
*   **Key Configurations:**
    *   Applied "Deny log on locally" and "Deny log on through Terminal Services" for the `GRP_ServiceAccounts` group.
*   **Why it matters:** Follows least-privilege best practices. Service accounts only need permissions to run background tasks, never to log in via a desktop or RDP.

---

## 6. Edge Browser Config & Department Drive Maps
**Goal:** Improve user experience and standardize the operating environment for standard users.
*   **Scope:** Linked to specific department OUs (e.g., `Users/IT`, `Users/Sales`).
*   **Key Configurations (Group Policy Preferences):**
    *   Mapped specific network drives based on department (e.g., Drive `I:` for IT, Drive `S:` for Sales) pointing to `FILE01`.
    *   Standardized Microsoft Edge startup pages and created a desktop shortcut for the "myITlab Portal".
*   **Why it matters:** Shows practical helpdesk/sysadmin skills related to daily user productivity and file server access.
