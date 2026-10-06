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
Active Directory is the identity backbone of most enterprise Windows environments. It answers one core question: who is allowed to do what? It controls which users can log into which machines, which groups can access which resources, and which policies apply to which parts of the organization.

This lab demonstrates an Active Directory Domain Services (AD DS) deployment on a Microsoft Azure Windows Server 2025 virtual machine, automated with PowerShell. It covers infrastructure provisioning, domain controller promotion, OU hierarchy design, role-based access control with security groups and user provisioning, Group Policy configuration, and automated audit validation. I also worked through the day-to-day tasks that are core to IT support and sysadmin work: password resets, account unlocks, and offboarding.

The same identity model of users, groups, role-based access, and centrally enforced policy carries over conceptually to cloud identity platforms like Microsoft Entra ID, which makes this lab foundational even for cloud-focused roles.

| | |
|---|---|
| **Platform** | Microsoft Azure |
| **OS** | Windows Server 2025 Datacenter |
| **Domain** | `lab.local` |
| **Automation** | PowerShell (`deploy-ad.ps1`) |

---

## 🎯 Business Context

Organizations rely on Active Directory as the backbone of identity and access management, centralizing how users are provisioned, granted access, and offboarded. This lab simulates standing up a new domain from scratch in a cloud environment—a task common in greenfield deployments, disaster recovery, and lab or development environments. Automating the provisioning pipeline reduces human error, enforces consistent OU and security group structures, and provides an auditable record of the environment’s configuration.

---

## ✅ Prerequisites

- Microsoft Azure account with permission to create a resource group, VMs, and a virtual network
- RDP client for connecting to the VMs
- PowerShell 5.1+ (7+ recommended) and VS Code (optional)
- Basic familiarity with networking (IP, DNS), PowerShell, and AD concepts (forest, domain, OU, GPO)
- A virtual network where the workstation's DNS points to the domain controller
- NSG rule allowing RDP (3389), restricted to your IP

---

## 🧩 Architecture

![Architecture Diagram](diagrams/architecture.svg)

| Resource | Name | Type |
|---|---|---|
| Resource Group | `rg-lab01-0626` | Azure Resource Group |
| Virtual Machine | `dc01` | `Microsoft.Compute/virtualMachine` |
| Virtual Network | `dc01-vnet` | Azure VNet |
| Network Security Group | `dc01-nsg` | Azure NSG |

---

## Steps

---

### 1. Deploy the Azure Virtual Machine

---

### 2. Connect via Remote Desktop (RDP)

Once the VM is running, retrieve its public IP from the Azure portal and connect using Remote Desktop.

---

### 3. Install AD DS and Group Policy Management Console

From an elevated PowerShell session on `dc01`, install the AD Domain Services role and the Group Policy Management Console (GPMC).

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature -Name GPMC
```

Both commands complete with `Success: True`.

![AD DS installation in progress](screenshots/Screenshot%202026-06-09%20221024.png)

![AD DS and GPMC installation successful](screenshots/Screenshot%202026-06-09%20223008.png)

---

### 4. Promote the Server to a Domain Controller

Run `Install-ADDSForest` to create a new forest and promote `dc01` to a domain controller for `lab.local`. The cmdlet handles DNS installation automatically.

```powershell
Install-ADDSForest -DomainName "lab.local" -InstallDNS ...
```

> **Note:** The DNS delegation warning is expected in isolated lab environments — no action required.

![Install-ADDSForest running — new forest installation and DNS setup](screenshots/Screenshot%202026-06-09%20223456.png)

---

### 5. Create Organizational Units (OUs)

Provision the department OU structure under `DC=lab,DC=local` to reflect the organization's business units.

```powershell
New-ADOrganizationalUnit -Name "IT"      -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"      -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"   -Path "DC=lab,DC=local"
```

> The `Computers` OU already exists by default — the error shown is expected and non-breaking.

![Automated OU provisioning — IT, Finance, HR, Sales](screenshots/Screenshot%202026-06-09%20224456.png)

---

### 6. Create Security Groups

Create role-based security groups inside their respective OUs to support least-privilege access control.

```powershell
New-ADGroup -Name "IT_Admins"      -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users"  -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"       -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"    -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

![Automated security group provisioning](screenshots/Screenshot%202026-06-09%20224544.png)

---

### 7. Create User Accounts

Bulk-provision user accounts with secure passwords, placing each user in their department OU.

| Username | OU | Display Name |
|---|---|---|
| `alice.chen` | IT | Alice Chen |
| `bob.patel` | Finance | Bob Patel |
| `carol.jones` | HR | Carol Jones |
| `david.smith` | Sales | David Smith |

![Batch user creation across IT, Finance, HR, and Sales OUs](screenshots/Screenshot%202026-06-09%20224601.png)

---

### 8. Assign Role-Based Group Memberships

Add each user to their corresponding security group to enable role-based access control (RBAC).

```powershell
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

![Role-based group membership assignment](screenshots/Screenshot%202026-06-09%20224623.png)

---

### 9. Create and Configure a Group Policy Object (GPO)

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

---

### 10. Run Automated Audit Validation

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

## Cleanup

To avoid ongoing Azure charges, deallocate or delete the `dc01` VM and associated resources from the `rg-lab01-0626` resource group when the lab is complete.

```bash
az group delete --name rg-lab01-0626 --yes --no-wait
```
