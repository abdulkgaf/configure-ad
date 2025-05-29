# configure-ad
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.

# On-premises Active Directory Deployed in the Cloud (Azure)

This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.

## Environments and Technologies Used
* Microsoft Azure (Virtual Machines/Compute)
* Remote Desktop
* Active Directory Domain Services
* PowerShell

## Operating Systems Used
* Windows Server 2022
* Windows 10 (21H2)

## High-Level Deployment and Configuration Steps
* Setup Resources in Azure
* Ensure Connectivity between the client and Domain Controller
* Install Active Directory
* Create an Admin and normal user account in AD

## Deployment and Configuration Steps

### Step 1: Create Resources in Azure

#### Create the Domain Controller VM (DC-1)
- Log into Azure Portal (portal.azure.com)
- Create a new Resource Group called "AD-Lab"
- Click "Create a resource" → "Virtual Machine"
- Configure Domain Controller:
  - Resource Group: AD-Lab
  - Virtual Machine Name: DC-1
  - Region: (US) East US 2
  - Image: Windows Server 2022 Datacenter: Azure Edition - Gen2
  - Size: Standard_B2s (2 vcpus, 4 GiB memory)
  - Username: labuser
  - Password: Cyberlab123!
  - Confirm password
  - Check licensing checkbox
- Click "Next: Disks" → "Next: Networking"
- Note the Virtual Network (Vnet) name created automatically
- Click "Review + create" → "Create"

#### Create the Client VM (Client-1)
- Create another Virtual Machine while DC-1 is deploying
- Configure Client VM:
  - Resource Group: AD-Lab (same as DC-1)
  - Virtual Machine Name: Client-1
  - Region: (US) East US 2 (same as DC-1)
  - Image: Windows 10 Pro, version 21H2 - Gen2
  - Size: Standard_B2ms (2 vcpus, 8 GiB memory)
  - Username: labuser
  - Password: Cyberlab123!
- Under Networking:
  - Virtual Network: Select the same Vnet as DC-1
  - Subnet: default (10.0.0.0/24)
- Click "Review + create" → "Create"

![Screenshot 2025-05-28 113215](https://github.com/user-attachments/assets/2289f017-3e6a-444c-b527-bd5bf6d5faea)

#### Set Domain Controller's NIC Private IP to Static
- Navigate to DC-1 VM → Networking → Network Interface
- Click on the Network Interface name
- Go to Settings → IP configurations
- Click on "ipconfig1"
- Change Assignment from "Dynamic" to "Static"
- Note the Private IP address (e.g., 10.0.0.4)
- Click "Save"

### Step 2: Ensure Connectivity Between Client and Domain Controller

![Screenshot 2025-05-28 113331](https://github.com/user-attachments/assets/76d02c42-d361-4a0e-be0e-740b929ffff3)

#### Test Initial Connectivity
- Remote Desktop into Client-1 using its public IP
- Login with labuser/Cyberlab123!
- Open Command Prompt
- Ping DC-1's private IP address: `ping 10.0.0.4`
- Notice the ping times out (this is expected due to firewall)

#### Enable ICMPv4 on Domain Controller
- Remote Desktop into DC-1 using its public IP
- Login with labuser/Cyberlab123!
- Open Windows Defender Firewall with Advanced Security
- Click "Inbound Rules" in left panel
- Find "Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)"
- Right-click both Private and Domain profile rules
- Click "Enable Rule" for both

#### Verify Connectivity
- Return to Client-1
- Try pinging DC-1 again: `ping 10.0.0.4`
- Ping should now succeed with replies
- Run `ping 10.0.0.4 -t` for continuous ping to verify consistent connectivity

### Step 3: Install Active Directory Domain Services

![Screenshot 2025-05-28 114850](https://github.com/user-attachments/assets/0813cc0d-afbe-49d9-80af-0b7ba2993d46)

#### Install AD DS Role on DC-1
- On DC-1, open Server Manager (should open automatically)
- Click "Add roles and features"
- Click "Next" through "Before You Begin"
- Installation Type: Role-based or feature-based installation
- Server Selection: Select DC-1 from server pool
- Server Roles: Check "Active Directory Domain Services"
- Click "Add Features" when prompted
- Click "Next" through Features, AD DS, and Confirmation
- Click "Install"
- Wait for installation to complete

#### Promote Server to Domain Controller
- In Server Manager, click the notification flag (warning triangle)
- Click "Promote this server to a domain controller"
- Deployment Configuration:
  - Select "Add a new forest"
  - Root domain name: mydomain.com
  - Click "Next"
- Domain Controller Options:
  - Forest functional level: Windows Server 2016
  - Domain functional level: Windows Server 2016
  - Check "Domain Name System (DNS) server"
  - DSRM Password: Password1
  - Confirm password: Password1
  - Click "Next"
- DNS Options: Click "Next" (ignore DNS delegation warning)
- Additional Options:
  - NetBIOS domain name: MYDOMAIN
  - Click "Next"
- Paths: Accept default paths, click "Next"
- Review Options: Click "Next"
- Prerequisites Check: Click "Install"
- Server will restart automatically

#### Reconnect After Restart
- Wait about 5 minutes for DC-1 to restart
- Remote Desktop back into DC-1
- Login as: mydomain.com\labuser
- Password: Cyberlab123!

### Step 4: Create Admin and Normal User Accounts in AD

![Screenshot 2025-05-28 115257](https://github.com/user-attachments/assets/75d644a8-0ec7-45e0-97ab-829b63f52dbd)

#### Create Organizational Units
- On DC-1, open Server Manager
- Click Tools → Active Directory Users and Computers
- Right-click mydomain.com → New → Organizational Unit
- Create "_EMPLOYEES" OU (uncheck "Protect container from accidental deletion")
- Create "_ADMINS" OU using the same process

#### Create Admin User Account
- Right-click "_ADMINS" OU → New → User
- First name: Jane
- Last name: Doe
- User logon name: jane_admin
- Click "Next"
- Password: Cyberlab123!
- Uncheck "User must change password at next logon"
- Check "Password never expires"
- Click "Next" → "Finish"

#### Add Admin User to Domain Admins Group
- Right-click Jane Doe → Properties
- Click "Member Of" tab
- Click "Add"
- Type "Domain Admins" → Click "Check Names"
- Click "OK" → "Apply" → "OK"

#### Create Normal User Account
- Right-click "_EMPLOYEES" OU → New → User
- First name: John
- Last name: Smith
- User logon name: john.smith
- Click "Next"
- Password: Cyberlab123!
- Uncheck "User must change password at next logon"
- Check "Password never expires"
- Click "Next" → "Finish"

### Step 5: Join Client-1 to Domain

![Screenshot 2025-05-28 122213](https://github.com/user-attachments/assets/b4c107e9-ee10-4193-8566-598e9d9c3463)

#### Configure Client-1 DNS Settings
- In Azure Portal, go to Client-1 → Networking
- Click on the Network Interface
- Go to DNS servers under Settings
- Select "Custom"
- Add DC-1's private IP address (10.0.0.4)
- Click "Save"
- Restart Client-1 from Azure Portal

![Screenshot 2025-05-28 122402](https://github.com/user-attachments/assets/7dc84c74-e06d-4398-bfc6-bb5def64bcc3)

#### Join Client-1 to Domain
- Remote Desktop into Client-1
- Login as original local user (labuser)
- Right-click "This PC" → Properties
- Click "Advanced system settings"
- Under Computer Name tab, click "Change"
- Select "Domain" and enter: mydomain.com
- Click "OK"
- Enter domain admin credentials:
  - Username: mydomain.com\jane_admin
  - Password: Cyberlab123!
- Click "OK" → "OK" → "Close"
- Restart Client-1 when prompted

### Step 6: Setup Remote Desktop for Non-Administrative Users

![Screenshot 2025-05-28 122716](https://github.com/user-attachments/assets/45e8691f-9982-4ea2-940c-ee8741c50a9d)

#### Configure Remote Desktop Access
- Remote Desktop into Client-1
- Login as: mydomain.com\jane_admin
- Right-click "This PC" → Properties
- Click "Advanced system settings"
- Under Remote Desktop, click "Select Users"
- Click "Add"
- Type "Domain Users" → "Check Names" → "OK"
- Click "OK" → "OK"

### Step 7: Test Domain Functionality

![Screenshot 2025-05-28 123216](https://github.com/user-attachments/assets/f4ed8066-a7d9-4469-b8da-bb96ac43e98d)

#### Test User Login
- Log out of Client-1
- Login as: mydomain.com\john.smith
- Password: Cyberlab123!
- Verify successful login

#### Verify Domain Join with PowerShell
- Open PowerShell as Administrator
- Run: `Get-ComputerInfo | Select-Object WindowsProductName, WindowsDomainName`
- Should show domain as "mydomain.com"

#### Create Additional Test Users (Optional)
- On DC-1, open PowerShell ISE as Administrator
- Run the following script to create multiple users:

```powershell
# Import AD Module
Import-Module ActiveDirectory

# Create 10 test users
For ($i=1; $i -le 10; $i++) {
    $Username = "user$i"
    $Password = ConvertTo-SecureString "Password123!" -AsPlainText -Force
    New-ADUser -Name $Username -UserPrincipalName "$Username@mydomain.com" -SamAccountName $Username -AccountPassword $Password -Enabled $true -Path "OU=_EMPLOYEES,DC=mydomain,DC=com"
    Write-Host "Created user: $Username"
}
```

#### Test Additional User Login
- On Client-1, logout and login as mydomain.com\user1
- Password: Password123!
- Verify successful domain authentication

## Deployment Complete!

You have successfully deployed a fully functional Active Directory environment in Azure with:

**Infrastructure Components:**
- Domain Controller (DC-1) running Windows Server 2022
- Client machine (Client-1) running Windows 10
- Proper network configuration and connectivity

**Active Directory Configuration:**
- Forest: mydomain.com
- Organizational Units for management
- Administrative and standard user accounts
- Domain-joined client computer
- Remote Desktop access for domain users

**Key Skills Demonstrated:**
- Azure Virtual Machine deployment and configuration
- Active Directory Domain Services installation and configuration
- DNS configuration and network troubleshooting
- User and organizational unit management
- Domain join procedures
- PowerShell automation for user creation

This lab environment provides a foundation for further Active Directory administration tasks and can be expanded with additional features like Group Policy, file shares, and more complex organizational structures.
