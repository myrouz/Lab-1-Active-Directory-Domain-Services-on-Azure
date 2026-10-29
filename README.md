# 🗄️ Lab 1: Active Directory Domain Services on Azure

![Platform: Microsoft Azure](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)
![Operating System: Windows Server 2025](https://img.shields.io/badge/Operating%20System-Windows%20Server%202025-0078D4?logo=windows&logoColor=white)
![Directory Service: Active Directory](https://img.shields.io/badge/Directory%20Service-Active%20Directory-0078D4?logo=microsoft&logoColor=white)
![Scripting Language: PowerShell](https://img.shields.io/badge/Scripting%20Language-PowerShell-5391FE?logo=powershell&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-Identity%20%26%20Access%20Management-6B4FA0)

---

## 🎥 Walkthrough

Learn how organizations manage users, computers, and access to network resources with Active Directory!

[Loom Video: Active Directory Domain Services Lab](https://www.loom.com/share/8dabc9d278f142ad891bd00f2dac9b9e)

---

## 📋 Overview
Active Directory Domain Services (AD DS) is a core component of many enterprise Windows environments, enabling centralized identity management, user authentication, group-based access control, and security policy enforcement.

This project demonstrates the deployment and configuration of an AD DS environment using a Windows Server 2025 virtual machine hosted in Microsoft Azure. The lab simulates a small enterprise environment in which IT administrators manage user identities, organize directory resources, assign permissions, enforce security policies, and automate administrative tasks using PowerShell.

Implementation includes provisioning the Azure virtual machine, installing AD DS, promoting the server to a Domain Controller, creating a domain, designing an Organizational Unit (OU) hierarchy, configuring security groups and user accounts, and implementing Group Policy Objects (GPOs).

I also used PowerShell to automate routine administrative tasks, including user provisioning, password resets, account unlocks, and account auditing. Configuration checks and audit scripts helped validate the environment and demonstrate practical Windows Server administration skills.

The project highlights core identity and access management principles, including centralized administration, role-based access control, user lifecycle management, and security policy enforcement. These skills provide a foundation for further work with Microsoft Entra ID and cloud identity management.

> Environment: This is a learning lab hosted in Microsoft Azure. It is intended for educational and portfolio purposes rather than production deployment.

---

## 🎯 Business Context

Organizations rely on Active Directory Domain Services (AD DS) as a core identity and access management platform for centralized user provisioning, authentication, resource access, and account lifecycle management. This lab simulates deploying a new Active Directory domain from the ground up in a Microsoft Azure environment, reflecting real-world scenarios such as greenfield infrastructure deployments, disaster recovery exercises, and lab or development environments.

Automating user provisioning and directory configuration with PowerShell helps reduce manual errors, standardize Organizational Unit (OU) structures and security group assignments, and improve configuration consistency and auditability. These practices support scalable identity administration, strengthen access control, and promote consistent security policy enforcement across enterprise Windows environments.

---

## 🧩 Architecture

<img width="1024" height="633" alt="AD DS-architecture" src="https://github.com/user-attachments/assets/b64dfe15-2bb0-40f9-a12d-8581599802bf" />

---
## 💻 Technology Stack 


| Category | Technology | Purpose in This Project |
|---|---|---|
| **Cloud Infrastructure** | Microsoft Azure Virtual Machines | Hosts the Windows Server virtual machine in the cloud. |
| | Azure Virtual Network (VNet) and Network Security Groups (NSGs) | Provides network connectivity for the VM and controls inbound and outbound traffic. |
| **Operating System and Directory Services** | Windows Server | Provides the operating system for the domain controller. |
| | Active Directory Domain Services (AD DS) | Centrally manages domain users, computers, authentication, and directory objects. |
| | Domain Name System (DNS) | Resolves domain names and helps domain-joined computers locate Active Directory services. |
| **Identity and Administration** | Active Directory Users and Computers (ADUC) | Creates and manages user accounts, groups, and Organizational Units. |
| | Active Directory Security Groups | Organizes users and helps manage access to resources through group membership. |
| | Organizational Units (OUs) | Organizes users, groups, and computers into containers for delegation and policy application. |
| | Group Policy Objects (GPOs) | Applies centralized configuration and security settings to users and computers. |
| | Group Policy Management Console (GPMC) | Creates, manages, and links Group Policy Objects. |
| **Automation and Remote Administration** | Windows PowerShell | Automates user provisioning, administration, and auditing tasks. |
| | Remote Desktop Protocol (RDP) | Provides remote access to administer the Windows Server VM. |


## ✅ Prerequisites


<details>
  <summary><b>1. Deploy a Windows Virtual Machine in Azure</b></summary>


a. **Search & Create:** Go to **Virtual machines** in the Azure Portal and click **Create > Azure virtual machine**.

b. **Basics Tab:**
  * Choose your subscription and resource group.
  * Enter a **Virtual machine name** (e.g., `dc01`).
  * Select your region and desired **Image** (e.g., Windows Server 2025).
  * Set up your administrator **Username** and **Password**.
  * Under **Public inbound ports**, check **RDP (3389)**.

c. **Review & Create:** Click **Review + create**, wait for validation, and click **Create**.

d. **Connect:** Once deployed, go to the resource, copy your **Public IP address**, and use RDP to log in.

<img width="1256" height="648" alt="Windows VM Deployed" src="https://github.com/user-attachments/assets/100423d3-1208-4297-8b7f-c6321af79196" />

</details>

<details>
  <summary><b>2. Connect via Remote Desktop (RDP)</b></summary>

a. **Connect** Click Go to Resource on the Deployment Complete page. Click Connect with dropdown. Select the first Connect option.

b. **Download RDP file** Select Download RDP file. Click file to open Windows Remote Desktop. 

c. **Authenticate** When prompted for credentials, enter the administrator **Username** and **Password** you created during the VM setup.

e. **Accept Certificate (If Prompted):** If a certificate warning appears regarding the remote computer's identity, click **Yes** or **Continue** to establish the connection.
</details>

---

## 🛠️ Hands-On Steps


<details>
  <summary><b>3. Install AD DS and Group Policy Management Console</b></summary>

Option 1: Server Manager (GUI)
 - Open Server Manager, then Manage → Add Roles and Features.
 - Choose Role-based or feature-based installation and select your server.
 - Check Active Directory Domain Services and accept the prompt to add the required features (management tools).
 - Click through to Install.

<img width="780" height="554" alt="AD DS Server Roles" src="https://github.com/user-attachments/assets/2886f480-e836-455a-903c-1fec9b019fea" />

> Check Active Directory Domain Services

<img width="775" height="552" alt="GPMC Server Role" src="https://github.com/user-attachments/assets/7f64acc8-e470-42f4-8fc9-3f3427ec6191" />

> Check Group Management Policy

<img width="780" height="552" alt="AD DS and GPMC Installation" src="https://github.com/user-attachments/assets/b673bf3c-2485-4ee2-a482-be5bc9978d11" />

> Click Install 

Option 2: Powershell (CLI)

From an elevated PowerShell session on `dc01`, install the AD Domain Services role and the Group Policy Management Console (GPMC).

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature -Name GPMC
```

Both commands complete with `Success: True`.

</details>

<details>
  <summary><b>4. Promote the Server to a Domain Controller</b></summary>

Option 1: Server Manager (GUI)

When installation finishes, click the flag icon in Server Manager and choose Promote this server to a domain controller to launch the configuration wizard.

 - **Deployment Configuration:** select **Add a new forest** and enter the root domain name (for example, `lab.local`).
 - **Domain Controller Options:** keep the default functional levels, leave **DNS server** and **Global Catalog** checked, and set a **Directory Services Restore Mode (DSRM)** password.
 - **DNS Options:** a warning that a delegation for the DNS server can't be created is expected for a new forest. Click **Next**.
 - **Additional Options:** confirm the NetBIOS domain name (for example, `LAB`).
 - **Paths:** accept the default locations for the database, log files and SYSVOL.
 - **Review Options:** confirm the settings. Optionally click **View script** to capture the equivalent PowerShell.
 - **Prerequisites Check:** wait for the checks to pass (warnings are fine, errors are not), then click **Install**.
 - `dc01` restarts automatically. Sign back in as `LAB\azureuser`.


<img width="1254" height="754" alt="Promote Domain Controller" src="https://github.com/user-attachments/assets/c4b57222-6362-42fe-9003-f8e7efb7044c" />

<img width="1254" height="754" alt="Add Forest" src="https://github.com/user-attachments/assets/22bb4b8d-3a2e-4141-a856-86b9ac9f07b1" />

<img width="1251" height="756" alt="Set DSRM Password" src="https://github.com/user-attachments/assets/8e74980c-6baa-402f-b77d-ad41a584fb5e" />

<img width="1250" height="754" alt="NetBIOS" src="https://github.com/user-attachments/assets/9c98b0bf-ed01-498b-860c-90b03bf1fe09" />

<img width="1248" height="754" alt="Installation" src="https://github.com/user-attachments/assets/8d670d55-41dc-4e04-a7d5-d9f7129d0a6a" />


Option 2: Powershell (CLI)

Run `Install-ADDSForest` to create a new forest and promote `dc01` to a domain controller for `lab.local`. The cmdlet handles DNS installation automatically.

```powershell
Install-ADDSForest -DomainName "lab.local" -InstallDNS ...
```
```
# Promote dc01 to the first domain controller in a new forest
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName "lab.local" `
  -DomainNetbiosName "LAB" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (Read-Host -AsSecureString "DSRM password") `
  -Force:$true
```
> **Note:** The DNS delegation warning is expected in isolated lab environments — no action required.

</details>

<details>
  <summary><b>5. Create Organizational Units (OUs)</b></summary>

Provision the department OU structure under `DC=lab,DC=local` to reflect the organization's business units.

```powershell
New-ADOrganizationalUnit -Name "IT"      -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"      -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"   -Path "DC=lab,DC=local"
```

> The `Computers` OU already exists by default — the error shown is expected and non-breaking.

<img width="1429" height="826" alt="Create OUs" src="https://github.com/user-attachments/assets/c8396f8e-64b4-4fdd-a225-b52b52a35e26" />

> OUs created with New-ADOrganizationalUnit and verified in Active Directory Users and Computers.

</details>

<details>
  <summary><b>6. Create Security Groups</b></summary>

Create role-based security groups inside their respective OUs to support least-privilege access control.

```powershell
New-ADGroup -Name "IT_Admins"      -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users"  -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"       -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"    -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

<img width="1405" height="848" alt="Create Security Group" src="https://github.com/user-attachments/assets/d9ac3624-ee7e-4edf-bd3f-d9161219edf8" />

> In Active Directory Users and Computers, select each OU and confirm it contains its security group with Type shown as Security Group - Global.

</details>

<details>
  <summary><b>7. Define Password/Create User Accounts/Assign Role-Based Group Memberships</b></summary>

Bulk-provision user accounts with secure passwords, placing each user in their department OU.

| Username | OU | Display Name |
|---|---|---|
| `alice.chen` | IT | Alice Chen |
| `bob.patel` | Finance | Bob Patel |
| `carol.jones` | HR | Carol Jones |
| `david.smith` | Sales | David Smith |


## IMPORTANT: Run this entire block together — not line by line

```powershell
# Step 1 — define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force
 
# Step 2 — create all 4 users (Name parameter is required)
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true
 
New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true
 
New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true
 
New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true
 
# Step 3 — add each user to their department group
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

<img width="1417" height="845" alt="Password:Users:Role" src="https://github.com/user-attachments/assets/84b612e6-437a-4b98-85bf-2588d3e96d14" />

> In Active Directory Users and Computers, select each OU and confirm it contains its user account (Type: User) and its department security group (Type: Security Group - Global), in this example bob.patel and Finance_Users in the Finance OU.


</details>

<details>
  <summary><b>8. Create and Configure a Group Policy Object (GPO)</b></summary>

Create the `IT Security Policy` GPO, link it to the IT OU, and inject baseline security registry settings — including inactivity screensaver enforcement — to meet corporate security requirements.

```powershell
Import-Module GroupPolicy
$GPO = New-GPO -Name "IT Security Policy" -Comment "Automated Corporate IT OU Baseline Security Configuration"
New-GPLink -Name "IT Security Policy" -Target "OU=IT,DC=lab,DC=local"

# Enforce screensaver activation
Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" `
  -ValueName "ScreenSaveActive" -Type String -Value "1"

# Require password on screensaver resume
Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" `
  -ValueName "ScreenSaverIsSecure" -Type String -Value "1"
```

<img width="787" height="599" alt="GPO created and Linked to OU=IT" src="https://github.com/user-attachments/assets/dd3e440f-8d7e-44d0-ba05-3ba338744860" />

> Creating and linking a GPO to the IT OU in Group Policy Management.


<img width="783" height="547" alt=" IT Security Policy" src="https://github.com/user-attachments/assets/ed37361d-6583-435e-bb31-72ed419aa5a0" />

> IT Security Policy GPO linked to the IT OU and shown as enabled in Group Policy Management.
</details>

<details>
  <summary><b>9. GPO Configuration: IT Security Policy</b></summary>  

Linked to the **IT** OU. All settings are under **Computer Configuration**.

| Setting | Value | Policy Path | Purpose |
|---------|-------|-------------|---------|
| Minimum password length | 12 | Windows Settings → Security Settings → Account Policies → Password Policy | Enforces strong passwords |
| Password must meet complexity requirements | Enabled | Windows Settings → Security Settings → Account Policies → Password Policy | Requires upper, lower, number, and symbol |
| Interactive logon: Machine inactivity limit | 900 seconds | Windows Settings → Security Settings → Local Policies → Security Options | Auto-locks the screen after 15 minutes |
| All Removable Storage classes: Deny all access | Enabled | Administrative Templates → System → Removable Storage Access | Prevents data exfiltration via USB drives |

<img width="885" height="700" alt="Password Policy Settings" src="https://github.com/user-attachments/assets/f90ee5e0-c621-4b6c-a95e-a753ce07ba58" />

> Set Minimum password length to 12 characters and enabled Password must meet complexity requirements in the IT Security Policy GPO. Together they reject short or predictable passwords by requiring length plus a mix of uppercase, lowercase, numbers, and symbols.

<img width="885" height="700" alt="Machine Inactivity Limit" src="https://github.com/user-attachments/assets/322879e1-d531-4e5f-adc9-c8c7d23fcdad" />

> Configured Interactive logon: Machine inactivity limit to 900 seconds. Workstations in the IT OU automatically lock after 15 minutes of inactivity, which protects unattended sessions.

<img width="885" height="700" alt="Removable Storage Access" src="https://github.com/user-attachments/assets/c6781eec-4a03-4ab7-923a-55e2241fc76b" />

> Enabled All Removable Storage classes: Deny all access. This blocks read and write access to USB drives, CDs/DVDs, and other removable media, reducing the risk of data exfiltration and malware introduced by USB.


</details>

<details>
  <summary><b>10. Common Help Desk Tasks</b></summary>

Day-to-day account management I practiced in the lab, using PowerShell.

| Scenario | Cmdlet |
|----------|--------|
| Reset a forgotten password | `Set-ADAccountPassword`, `Set-ADUser` |
| Unlock a locked-out account | `Unlock-ADAccount` |
| Offboard an employee | `Disable-ADAccount` |
| Audit inactive accounts and group membership | `Get-ADUser`, `Get-ADPrincipalGroupMembership` |

### Reset a password

The most common help desk ticket. Always force a password change at next logon so the user sets their own password immediately.

```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

> The password above is a lab-only temporary value.

### Unlock a locked account

Accounts lock automatically after too many failed sign-in attempts, which makes this one of the most frequent calls to any help desk.

```powershell
Unlock-ADAccount -Identity "carol.jones"
```

To see which accounts are currently locked:

```powershell
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName
```

### Disable an account (offboarding)

When someone leaves the organization, disable the account instead of deleting it. Disabling preserves the account history and group memberships for audit purposes. Deletion is permanent.

```powershell
Disable-ADAccount -Identity "david.smith"

# Find all currently disabled user accounts
Search-ADAccount -AccountDisabled -UsersOnly | Select-Object Name, SamAccountName
```

### Audit and reporting

Security and compliance teams regularly need reports on inactive accounts, group memberships, and sign-in activity.

```powershell
# Enabled accounts with no logon in 90 days (or that have never logged on)
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter 'Enabled -eq $true' -Properties LastLogonDate |
  Where-Object { $_.LastLogonDate -lt $cutoff -or -not $_.LastLogonDate } |
  Select-Object Name, LastLogonDate

# Group membership for a specific user
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

</details>

<details>
  <summary><b>11. Verification — Confirm the Lab is Working</b></summary>

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Domain controller is running | `Get-ADDomainController` | Returns DC info for `dc01`, domain and forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter * \| Select-Object Name` | Lists IT, Finance, HR, Sales, and the built-in Domain Controllers OU |
| Users exist and are enabled | `Get-ADUser -Filter 'Enabled -eq $true' -SearchBase "DC=lab,DC=local" \| Select-Object SamAccountName` | Includes alice.chen, bob.patel, carol.jones, david.smith |
| Group memberships are correct | `Get-ADGroupMember -Identity IT_Admins` | Returns alice.chen (repeat for the other three groups) |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows IT Security Policy under GpoLinks, Enabled: True |

<img width="1211" height="843" alt="Confirm DC is running" src="https://github.com/user-attachments/assets/302ad518-90f9-4d24-94db-9f6edd00c30a" />

> Shows the status and details of the main server running the network (lab.local). It displays its IP address, operating system (Windows Server 2025), and confirms it is handling all core management duties.

<img width="1211" height="843" alt="Verify OUs exist" src="https://github.com/user-attachments/assets/727f07df-f2e9-4731-9b35-06bb6127bec9" />

> Lists all the main organizational folders (OUs) set up in the network directory.

<img width="1211" height="843" alt="Veriy users exist and are enabled" src="https://github.com/user-attachments/assets/95a8c2f1-6b0c-4298-9414-c6c93b1d8ff4" />

> Pulls up a list of all active, working user accounts on the network, showing usernames.

<img width="1210" height="840" alt="Verify Group Membership" src="https://github.com/user-attachments/assets/c491c363-e4ab-4ed3-a2ed-6efa39163fb2" />

> Lists who belongs to the IT Administrators group. 

<img width="1206" height="838" alt="GPO is Linked" src="https://github.com/user-attachments/assets/8d3d2788-8954-4a0a-84a1-3854ce1bc805" />

> Checks the security rules applied to the IT department. It confirms that the department is correctly receiving its security settings and the standard domain rules.
  
</details>
  
---

## 🛠️ Key Skills Demonstrated

**Cloud Infrastructure & Remote Administration:** Provisioning and managing virtual machines within Microsoft Azure using Remote Desktop Protocol (RDP) with customized local resource mapping.

**Domain Controller Deployment & Management:** Promoting a Windows Server to an Active Directory Domain Controller (lab.local), establishing a new AD forest, and managing core DNS and identity services

**Directory Structure Design:** Creating and organizing Organizational Units (OUs) to segment departmental resources (IT, Finance, HR, Sales) and computer accounts.

**Identity & Access Management (IAM):** Creating, enabling, and managing user accounts alongside configuring security groups to implement role-based access control (RBAC) and the principle of least privilege.

**Group Policy Administration:** Deploying and configuring Group Policy Objects (GPOs) via the Group Policy Management Console (GPMC) to enforce enterprise-wide security baselines—such as password complexity requirements, screen lock timers, and USB storage restrictions.

**IT Help Desk Operations:** Performing routine administrative tasks including resetting passwords, forcing password changes at next logon, unlocking locked user accounts, and managing the employee account lifecycle (disabling accounts during offboarding).

**PowerShell Automation & Auditing:** Using Active Directory PowerShell cmdlets (Get-ADUser, Get-ADGroupMember, Get-GPInheritance, etc.) to script administrative tasks, audit group memberships, query inactive accounts, and verify domain health.

---

## 📖 Key Terms

| Term | Description & Role in Active Directory |
| --- | --- |
| **Forest** | The top-level container of the entire Active Directory structure, representing the organization itself.|
| **Domain Controller (DC)** | A core server running Active Directory that acts as the "brain" of the identity system, handling authentication requests and managing domain DNS. |
| **Organizational Unit (OU)** | A directory folder used to organize users, computers, and groups by department or function, allowing policies to be applied centrally. |
| **Security Group** | A container holding user accounts to grant access to resources collectively, implementing efficient role-based access control. |
| **Group Policy Object (GPO)** | A collection of configuration settings functioning as an automated rulebook enforced across users or computers within a linked OU. |
| **User Account** | An entry representing a person or service, containing credentials, group memberships, and attributes that control access to network resources. |

---

## 💡Lessons Learned 

