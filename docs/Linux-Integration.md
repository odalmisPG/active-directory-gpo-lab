<div align="center">
  <h1>🔐 Enterprise Linux Integration with Active Directory (SSSD & Realmd)</h1>
  <p>
    <img src="https://img.shields.io/badge/Fedora-2F4153?style=for-the-badge&logo=fedora&logoColor=white" alt="Fedora Linux" />
    <img src="https://img.shields.io/badge/Windows_Server-0078D6?style=for-the-badge&logo=windows&logoColor=white" alt="Windows Server" />
    <img src="https://img.shields.io/badge/Active_Directory-0078D6?style=for-the-badge&logo=microsoft&logoColor=white" alt="Active Directory" />
    <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="Bash" />
  </p>
</div>

<hr>

## 📌 Project Overview
**Objective:** Implement centralized identity management by integrating a Fedora Linux workstation (`GAN-WS-003`) into a Windows Server Active Directory domain (`myITlab.local`). 

**Outcome:** Successfully joined the Linux machine to the domain, enabled Active Directory users to authenticate using their domain credentials, and enforced Windows Group Policy Objects (GPOs) to strictly control Linux login access based on AD group membership.

---

## ⚙️ Implementation Steps

### 1. Domain Join & Core Configuration
Utilized `realmd` to discover the Active Directory domain and seamlessly join the Fedora workstation. Configured the SSSD daemon to map Windows user attributes to Linux POSIX attributes automatically.

```bash
# Example commands used for domain join and configuration
sudo realm discover myITlab.local
sudo realm join myITlab.local -U Administrator
```
### 2. Automated Workspace Provisioning
Enabled PAM (Pluggable Authentication Modules) to automatically generate a /home/user directory for network accounts upon their first local login.

```bash
sudo authselect enable-feature with-mkhomedir
```
### 3. Implementing GPO-Based Access Control
Configured /etc/sssd/sssd.conf to respect Windows Group Policy by enforcing ad_gpo_access_control. Created a dedicated Active Directory security group (GRP-LinuxAdmins) and configured Windows GPO User Rights Assignment (Allow log on locally and Allow log on through Remote Desktop Services) to specifically permit the admin group while denying standard users.

## 🛠️ Advanced Troubleshooting & Problem Solving

During the integration, I encountered a complex cross-platform authentication block where valid AD users consistently received "Access Denied" errors upon login. Below is the workflow I used to diagnose and resolve the issue.

<details>
<summary><b>🚨 "Access Denied" Login Failure & GPO Parsing Remediation</b></summary>
<br>

**The Problem:** 
An Active Directory user (`vanessa.hansen-wa@myitlab.local`) was placed in the correct `GRP-LinuxAdmins` security group and granted the "Allow log on locally" right via a Linux-specific GPO. However, the user was continuously rejected at the Fedora login screen with a default "Access Denied" message despite SSSD being correctly joined to the domain.

**The Diagnostic Process:**
1. **Ruling out Stale Cache (SID Mismatch):** My initial hypothesis was that SSSD had cached an old Security Identifier (SID) because the AD user account had been recently deleted and recreated. To eliminate this variable, I stopped the SSSD service, systematically purged the internal SSSD database (`/var/lib/sss/db/*`), destroyed all Kerberos tickets (`kdestroy -A`), forced a fresh AD sync (`sss_cache -E`), and restarted the service. 
2. **Deep Log Analysis:** When the cache clear did not resolve the issue, I redirected the SSSD GPO trace logs into a text file for deep analysis (`sudo grep -i "gpo" /var/log/sssd/sssd_myitlab.local.log > ~/gpo_results.txt`). 

**The Root Cause:** 
By analyzing the log output, I discovered the true cause. A higher-level Windows Service Account GPO (`ComputersDeny-ServiceAccount-Logon`) applied to the parent OU contained Windows-specific `.inf` formatting. When the Linux machine attempted to read this policy, it crashed the Linux `.ini` text parser (`iniconfigparse failed 5` / `Error 8 on line 1 Failed to read line`). When the SSSD GPO parser fails to read any policy, it "fails safe" and defaults to blocking all user logins.

**The Solution:** 
To fix the parser crash without modifying the structure of the parent Windows policy, I utilized Advanced Security Settings in AD Group Policy Management. 
1. In the AD directory picker, I exposed the hidden Computer objects.
2. I added the Linux machine's computer object (`GAN-WS-003$`) to the Delegation list of the incompatible Service Account GPO.
3. I applied a strict **Deny: Apply group policy** rule specifically for that Linux machine. 
4. After running a final `sss_cache -E` on the Fedora machine, SSSD bypassed the crashing GPO, successfully parsed the Linux login GPO, and full authentication functionality was restored.
</details>

## 💻 Common SSSD/Fedora Commands Used

| Command | Description | Example Usage |
| :--- | :--- | :--- |
| `ls -l [path]` | Lists directory files to find specific log filenames. | `sudo ls -l /var/log/sssd/` |
| `grep -i [text] [file]` | Searches for specific text (case-insensitive) in a file. | `sudo grep -i "gpo" /var/log/sssd/sssd_myitlab.local.log` |
| `grep [text] > [file]` | Redirects output to a text file instead of the screen. | `sudo grep -i "gpo" ... > ~/results.txt` |
| `sss_cache -E` | Clears the SSSD cache, forcing a refresh from AD. | `sudo sss_cache -E` |
| `kdestroy -A` | Destroys all Kerberos tickets for a clean state. | `sudo kdestroy -A` |
| `id [username]` | Displays a user's UID, GID, and AD groups. | `id vanessa.hansen-wa@myitlab.local` |
| `realm list` | Shows Active Directory domain details. | `realm list` |
<br>

### ✅ Validation & Results
The integration was successfully validated by logging into the Fedora workstation as the Active Directory user vanessa.hansen-wa@myitlab.local, who was granted restricted login access via GPO and mapped to local sudo privileges via the /etc/sudoers configuration.

The screenshot below demonstrates the successful domain join (realm list) and verifies that the Linux system is successfully pulling the user's UID, GID, and Active Directory group memberships via SSSD (id).

<div align="center"> <img src="../images/linux-ad-integration.png" alt="Linux AD Integration Success Screenshot" width="800"> <br> <i>Terminal output confirming domain status and successful AD user authentication.</i> </div> 
