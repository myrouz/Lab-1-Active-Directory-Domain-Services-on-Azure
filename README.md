# Lab 1: Active Directory Domain Services on Azure

![Platform: Microsoft Azure](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)
![Operating System: Windows Server 2025](https://img.shields.io/badge/Operating%20System-Windows%20Server%202025-0078D4?logo=windows&logoColor=white)
![Directory Service: Active Directory](https://img.shields.io/badge/Directory%20Service-Active%20Directory-0078D4?logo=microsoft&logoColor=white)
![Scripting Language: PowerShell](https://img.shields.io/badge/Scripting%20Language-PowerShell-5391FE?logo=powershell&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-Identity%20%26%20Access%20Management-6B4FA0)

---

## 🎥 Walkthrough

Follow along as I complete this lab! 

https://www.loom.com/share/8dabc9d278f142ad891bd00f2dac9b9e

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

<img width="617" height="378" alt="Add Forest" src="https://github.com/user-attachments/assets/22bb4b8d-3a2e-4141-a856-86b9ac9f07b1" />

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

![Automated OU provisioning — IT, Finance, HR, Sales](screenshots/Screenshot%202026-06-09%20224456.png)
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

![Automated security group provisioning](screenshots/Screenshot%202026-06-09%20224544.png)
</details>

<details>
  <summary><b>7. Create User Accounts</b></summary>

Bulk-provision user accounts with secure passwords, placing each user in their department OU.

| Username | OU | Display Name |
|---|---|---|
| `alice.chen` | IT | Alice Chen |
| `bob.patel` | Finance | Bob Patel |
| `carol.jones` | HR | Carol Jones |
| `david.smith` | Sales | David Smith |

![Batch user creation across IT, Finance, HR, and Sales OUs](screenshots/Screenshot%202026-06-09%20224601.png)
</details>

<details>
  <summary><b>8. Assign Role-Based Group Memberships</b></summary>

Add each user to their corresponding security group to enable role-based access control (RBAC).

```powershell
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

![Role-based group membership assignment](screenshots/Screenshot%202026-06-09%20224623.png)
</details>

<details>
  <summary><b>9. Create and Configure a Group Policy Object (GPO)</b></summary>

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

![GPO created and linked to OU=IT — IT Security Policy](screenshots/Screenshot%202026-06-09%20225758.png)

![ScreenSaveActive registry policy injected](screenshots/Screenshot%202026-06-09%20225824.png)

![ScreenSaverIsSecure registry policy injected](screenshots/Screenshot%202026-06-09%20225845.png)
</details>

<details>
  <summary><b>10. Run Automated Audit Validation</b></summary>

Execute the built-in audit report to verify the full deployment: domain controller status, OU hierarchy, user accounts, group nesting, and GPO inheritance.

**Domain Controller:**

| ComputerName | Operating System | Forest |
|---|---|---|
| `dc01` | Windows Server 2025 Datacenter | `lab.local` |

**OU Structure verified:** IT, Finance, HR, Sales  
**Users verified:** alice.chen, bob.patel, carol.jones, david.smith (all enabled)  
**Group nesting verified:** alice.chen → IT_Admins  
**GPO inheritance verified:** IT Security Policy → OU=IT

![Audit report — DC status and OU hierarchy](screenshots/Screenshot%202026-06-09%20230304.png)

![Audit report — user validation and group nesting](screenshots/Screenshot%202026-06-09%20230534.png)

![Audit report — GPO inheritance confirmed on IT OU](screenshots/Screenshot%202026-06-09%20230702.png)
</details>
  
---

## Key Skills Demonstrated

- Azure VM provisioning and NSG configuration
- PowerShell automation of AD DS installation and forest promotion
- Organizational unit (OU) hierarchy design aligned to business structure
- Role-based security group provisioning
- Bulk user account creation with secure credential handling
- Group Policy Object (GPO) creation, linking, and registry-based policy injection
- Automated audit scripting with `Get-ADDomainController`, `Get-ADOrganizationalUnit`, `Get-ADUser`, `Get-ADGroupMember`, and `Get-GPInheritance`
- Git version control for infrastructure scripts

---

## Vocabulary

---

## Lessons Learned 

