# Active-Directory-Infrastructure-Automation
Automated deployment configuration lab simulating an enterprise identity control plane for Huntsville Defense Solutions
Enterprise Infrastructure Automation: Active Directory Identity Pipeline

# Enterprise Identity Automation: Active Directory Infrastructure

## Executive Summary
* **Client:** Huntsville Defense Solutions (Aerospace & Defense Contractor)
* **Objective:** Deploy an isolated Windows Server 2022 Domain Controller and execute an **Infrastructure-as-Code (IaC)** pipeline to provision 100 corporate user identities, departmental security boundaries, and enforced credential policies in under 60 seconds.

---

## Technical Execution & Key Learnings

### **Phase A & B: Virtual Sandbox & Hardware Provisioning**
* **Execution:** Initialized an isolated VirtualBox hypervisor sandbox and assigned 4GB RAM, 2 CPUs, 50GB VHD, and locked the adapter to an **Internal Network** (`Huntsville-Corp-Net`).
* **Key Takeaway:** Learned how air-gapped virtual networks isolate test domains and prevent IP/broadcast conflicts on physical host networks.

### **Phase C: OS Installation**
* **Execution:** Deployed Windows Server 2022 Standard Evaluation (Desktop Experience) and configured the master Administrator credentials.
* **Key Takeaway:** Mastered enterprise server OS deployment baselines and GUI vs. Server Core trade-offs.

### **Phase D: Static IP & Network Routing**
* **Execution:** Assigned static IPv4 routing (`192.168.10.10/24`) and pointed Preferred DNS directly to loopback identity routing (`192.168.10.10`).
* **Key Takeaway:** Understood why Domain Controllers require fixed IP anchors and self-referencing DNS to maintain domain locator services across an enterprise network.

> <img width="852" height="766" alt="Case Study-VM_PhaseD" src="https://github.com/user-attachments/assets/6925cca7-5852-4997-97db-c76c7abb9e68" />


### **Phase E: Active Directory Forest Promotion**
* **Execution:** Installed Active Directory Domain Services (AD DS) and promoted the standalone node to root Domain Controller for `huntsvilledefense.local`.
* **Key Takeaway:** Gained hands-on experience with domain promotion mechanics, Schema creation, and Directory Services Restore Mode (DSRM) safeguards.

<img width="865" height="736" alt="Screenshot 2026-09-28 114158" src="https://github.com/user-attachments/assets/25756678-9142-4e9b-83e7-0d0bed9ec13a" />
> <img width="908" height="755" alt="Screenshot 2026-09-28 114801" src="https://github.com/user-attachments/assets/bbefc0d7-f544-4a9e-be35-7a770b1fb852" />


### **Phase F: PowerShell Identity Automation Pipeline**
* **Execution:** Created departmental Organizational Units (`Engineering`, `Finance`, `Human_Resources`), populated `C:\lab-users.csv` with 100 employee profiles, and executed a custom PowerShell automation script to parse the roster and provision all accounts instantly.
* **Key Takeaway:** Learned how raw business data (HR CSV exports) connects to administrative scripting using string manipulation (`Substring`, `ToLower`), loops (`foreach`), and parameter passing into `New-ADUser` cmdlets.

><img width="911" height="729" alt="Screenshot 2026-09-28 121350" src="https://github.com/user-attachments/assets/7f639317-8ba1-4bb7-a2ef-342d253a462c" />
<img width="400" height="400" alt="Screenshot 2026-09-28 121600" src="https://github.com/user-attachments/assets/ad1a05c8-fae8-49ae-9fd3-280e531214c7" /> <img width="400" height="400" alt="Screenshot 2026-09-28 121618" src="https://github.com/user-attachments/assets/290c2340-04e8-4aa5-bd15-97a3eb24ef62" /> <img width="400" height="400" alt="Screenshot 2026-09-28 121632" src="https://github.com/user-attachments/assets/c0123ba9-5c32-458b-8cea-4b394ae1d34f" />



---

## PowerShell Onboarding Script

```powershell
# Import corporate CSV database
$Users = Import-Csv -Path "C:\lab-users.csv"

foreach ($User in$Users) {
    # Generate standardized username (First Initial + Last Name)
    $Username = ($User.FirstName.Substring(0,1) +$User.LastName).ToLower()
    
    # Map target Organizational Unit
    $TargetOU = "OU=$($User.Department),DC=huntsvilledefense,DC=local"
    
    # Set uniform secure password requiring reset at first login
    $SecurePassword = ConvertTo-SecureString "HuntsvilleTemp2026!" -AsPlainText -Force
    
    # Provision account in Active Directory
    New-ADUser `
        -Name "$($User.FirstName) $($User.LastName)" `
        -SamAccountName $Username `
        -UserPrincipalName "$Username@huntsvilledefense.local" `
        -GivenName $User.FirstName `
        -Surname $User.LastName `
        -Title $User.Title `
        -Path $TargetOU `
        -AccountPassword $SecurePassword `
        -ChangePasswordAtLogon $true `
        -Enabled $true

    Write-Host "Successfully onboarded user: $Username into$TargetOU" -ForegroundColor Green
}<img width="948" height="766" alt="Case Study-VM_PhaseD" src="https://github.com/user-attachments/assets/6f621d91-5dbb-42d8-ac21-cbed58a8fc5e" />
<img width="948" height="766" alt="Case Study-VM_PhaseD" src="https://github.com/user-attachments/assets/b7ee8849-f86c-47ea-983e-4361c7ea8ef2" />
