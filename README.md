# ActiveDirectory-Homelab
Abdullah’s Active Directory Home Lab is a virtualized, self-contained environment built for cybersecurity learning and testing. This lab is designed to help develop hands-on skills in Windows system administration, domain configuration, network management, and offensive security techniques like lateral movement and Active Directory exploitation.
---

**The environment consists of:**
- A Windows Server 2022 Domain Controller configured with Active Directory, DNS, and core services
- A Windows 11 Client machine joined to the domain for testing authentication, GPOs, and user management
- A Kali Linux Attacker machine used to perform ethical hacking techniques, AD enumeration, and privilege escalation
---

## 🧱 Lab Topology

![image](https://github.com/user-attachments/assets/0735b215-b1f7-47cc-8f48-17276dfc6545)

| Role              | Hostname     | IP Address       | OS                     |
|-------------------|--------------|------------------|------------------------|
| Domain Controller | Abdullah_DC        | 192.168.10.10     | Windows Server 2022   |
| AD Client         | CLIENT01    | 192.168.10.20     | Windows 11 Pro        |
| Attacker          | KALI         | 192.168.10.150    | Kali Linux Rolling    |


## ⚙️ Lab Setup

- **Hypervisor**: VMware Workstation 17.x 
- **Networking**: Host-Only + NAT (for Kali)
- **OS Images**:
  - Windows Server 2022 ISO
  - Windows 11  ISO
  - Kali Linux  ISO
---
## 🖥️ Domain Controller Configuration
This section explains how I installed and configured Active Directory Domain Services (AD DS) and DNS on a Windows Server 2022 machine to promote it as the domain controller for the Abdullah-AD.local domain.
---
### 🔧 Server Basics
- Hostname: Abdullah_DC
- IP Address: 192.168.10.10
- OS: Windows Server 2022 (Desktop Experience)
- Role: Domain Controller + DNS Server
---
![image](https://github.com/user-attachments/assets/f4f59b00-bcbc-4d24-a27c-dd8d4a80c9f7)

Static IP configuration

Open `Ethernet0` → `Properties` → `Internet Protocol Version 4 (TCP/IPv4)` → `Properties` and fill in:

| Setting | Value |
|---------|-------|
| IP address | 192.168.10.10 |
|Subnet mask | 255.255.255.0 |
| Default gateway | 192.168.10.1 |
| Preferred DNS server | 127.0.0.1|

![image](https://github.com/user-attachments/assets/01db936d-8795-4ecc-87c3-ab697438e452)

----
### Install AD DS & DNS
----
**Selecting `Add Roles and Features` to begin installing AD DS and DNS roles.**
![image](https://github.com/user-attachments/assets/03bf1373-982d-4ff0-b1f8-409087e0b255)


**Role Selection screen `Active Directory Domain Services` and `DNS Server` are checked here.**
![image](https://github.com/user-attachments/assets/226568b9-fd3f-4487-b6c0-75a71628d79d)


Choosing `Add a new forest` and entering the domain name: `Abdullah-AD.local`. This creates your top-level AD forest.

![image](https://github.com/user-attachments/assets/935a9311-ff86-40ab-9db9-3ccd4dd0288e)


NetBIOS domain name auto-populated as `ABDULLAH-AD`. This will be used in domain logins like `ABDULLAH-AD\Administrator`.

![image](https://github.com/user-attachments/assets/586483f8-5f3a-4e08-88a8-39e46f7a4953)


![image](https://github.com/user-attachments/assets/f4496fc1-37c9-4568-b008-5ddb0ce2f55b)

----
### Create OUs & Users
You can use either the Active Directory Users and Computers (ADUC) GUI or PowerShell. Example PowerShell snippet:
----
**Launch Active Directory Users and Computers (ADUC) **

![image](https://github.com/user-attachments/assets/19c57002-a5f9-4808-b41d-1090bce14636)


#### Manually adding OU and adding users

![image](https://github.com/user-attachments/assets/777c7ab6-6f6a-4c58-8997-9f7511ac5dd1)
---
#### Using Powershell
---
```bash
import-Module ActiveDirectory
```

![image](https://github.com/user-attachments/assets/2e1edcc1-3182-4d93-92b0-e4fb7ad664e1)

**Adding a single user**
```bash
New-ADUser -Name "John Doe"
-GivenName "John' -Surname "Doe"
-SamAccountName "jdoe"
-UserPrincipalName "jode@abdullah-AS.local'
-AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force)
-Enabled $true
-Path "CN=Users, DC=Abdullah-AD, DC=local"
```

![image](https://github.com/user-attachments/assets/7142d0a3-c04f-450f-80dd-60704f6a2c60)

**Create a Bulk of Users**

```bash
$users = @(
    @{Name="John Doe"; Username="jdoe"; Password="Password123!"},
    @{Name="Jane Smith"; Username="jsmith"; Password="Password123!"},
    @{Name="Alice Admin"; Username="alice"; Password="AdminPass123!"},
    @{Name="sqlsvc Service Account"; Username="sqlsvc"; Password="Service123!"},
    @{Name="ASREP Roasting"; Username="asrep"; Password="WeakPass123!"}
)

foreach ($u in $users) {
    New-ADUser -Name $u.Name `
        -SamAccountName $u.Username `
        -UserPrincipalName "$($u.Username)@Abdullah-AD.local" `
        -AccountPassword (ConvertTo-SecureString $u.Password -AsPlainText -Force) `
        -Enabled $true `
        -Path "OU=CompanyUsers,DC=Abdullah-AD,DC=local"
```

![image](https://github.com/user-attachments/assets/d9cd79c1-5665-41c1-afe6-268a7a35cd4c)

## 🖥️ AD Client Configuration
This section outlines the configuration of the Windows 11 client machine and its successful integration into the Active Directory domain Abdullah-AD.local.
---
### 💻 Client Details
- Hostname: CLIENT01
- IP Address: 192.168.10.20
- OS: Windows 11 Pro
- Network: Static IP on the same subnet as the Domain Controller (192.168.10.0/24)
- DNS: Points to DC (192.168.10.10) to resolve the domain name
----
### Step-by-Step Setup 

Navigate to System Properties to change the computer name and domain settings. 
- This is accessed via `Control Panel` > `System`

Clicking `Change settings` and setting the hostname to CLIENT01. This helps identify the machine within the domain.
In the `Domain` field, enter `Abdullah-AD.local` to begin joining the client to the AD environment.

![image](https://github.com/user-attachments/assets/b1bc41e7-0e17-40ca-8e76-9d1d131df6a1)

Prompt appears requesting credentials, this is where you enter `Administrator` and the domain `admin password`.

![image](https://github.com/user-attachments/assets/65c60a44-06c4-4632-9eb6-2801e8d4e90a)


Reboot, then log in for any user you created

![image](https://github.com/user-attachments/assets/6f31ae84-edf9-4125-9de5-61041c075405)

Verify with `whoami` that you should see the domain prefix.

![image](https://github.com/user-attachments/assets/7b147535-c608-4cf3-9b89-547153c236e8)

Success! The machine is now joined to the domain. You receive a confirmation message and must restart the computer.

