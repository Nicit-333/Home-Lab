# Home Lab Part 2: Domain Login & Group Policy Management

![Group Policy Management](images/GroupPolicy-Management.jpg)

## Introduction
This guide continues our Active Directory lab setup, focusing on troubleshooting domain login issues and implementing Group Policy Objects (GPOs) for centralized management of our Windows 10 client (Client1).

## Step 1 - Troubleshooting Domain Login Failure

### Problem Summary
- Client1 (Windows 10, domain-joined) failed to authenticate domain users
- Client1 was missing from the "Computers" container in Active Directory

### Resolution Steps
1. **Verified user account** in Active Directory:
   - Confirmed credentials and domain format were correct
2. **Checked AD "Computers" container**:
   - Client1 was not listed, indicating improper domain join
3. **On Client1 login screen**:
   - Selected "Other user" → Domain name appeared correctly
   - Logged in using local administrator account

### Solution Implementation
1. **Disjoined Client1 from domain**:
   - Settings > System > About > Rename this PC (advanced)
   - Changed to workgroup and restarted
2. **Rejoined Client1 to domain**:
   - Repeated steps, entered domain: mydomain.com
   - Provided domain admin credentials when prompted
   - Restarted machine
3. **Verification**:
   - Confirmed Client1 now appeared in AD "Computers" container
   - Successfully logged in with domain user account

### Root Cause Analysis
- Likely causes:
  - Incomplete/corrupted initial domain join
  - Broken secure channel between Client1 and DC

![AD Computers Container](images/AD-Computers-Container.png)

## Step 2 - Group Policy Management Setup

### Install Group Policy Management
1. Open **Server Manager**
2. Go to **Manage > Add Roles and Features**
3. Navigate to **Features > Group Policy Management**
4. Click **Install**

### Create Organizational Unit (OU)
1. Open **Group Policy Management Console**
2. Right-click domain → **New Organizational Unit**
3. Name: **TestComputers**
4. Moved Client1 from default "Computers" container to new OU

![GPMC Console](images/GPMC-Console.png)

## Step 3 - Implement Desktop Wallpaper Policy

### Shared Folder Setup
1. Created folder: `C:\Wallpapers`
2. Added wallpaper image: `ilovehousemusic.jpg`
3. Configured sharing:
   - Share name: Wallpapers
   - Permissions:
     - Removed "Everyone"
     - Added Domain Computers (Read access)
   - UNC Path: `\\DC\Wallpapers\ilovehousemusic.jpg`

### GPO Creation
1. In GPMC, right-click **TestComputers OU** → **Create a GPO**
2. Named: **Set Desktop Wallpaper**
3. Configured:
   - User Configuration > Policies > Admin Templates > Desktop > Desktop Wallpaper
   - Enabled policy
   - Set wallpaper path to UNC
   - Style: Fill

### Troubleshooting
- Initial failure due to permissions → Added Authenticated Users
- Policy not applying → Enabled Loopback Processing (Merge mode)
- Final verification: Wallpaper applied after `gpupdate /force` and reboot

## Step 4 - Implement USB Restriction Policy

### GPO Configuration
1. Created new GPO named **Disable USB Storage**
2. Configured:
   - Computer Configuration > Policies > Admin Templates > System > Removable Storage Access
   - Enabled:
     - Deny read access
     - Deny write access

### Testing
- Confirmed:
  - USB storage devices blocked
  - HID devices (mice/keyboards) still functional

![USB Policy](images/USB-Policy.png)

## Step 5 - Deploy Chrome via GPO

### Preparation
1. Downloaded Chrome MSI from [Chrome Enterprise](https://chromeenterprise.google/browser/download/)
   - Fixed DNS issue by adding 8.8.8.8 as alternate DNS
2. Created shared folder:
   - Path: `C:\Software\Chrome`
   - Shared as `\\DC\Software` with Read access for:
     - Domain Computers
     - Authenticated Users

### GPO Implementation
1. Created new GPO named **Deploy Chrome**
2. Configured:
   - Computer Configuration > Policies > Software Settings > Software Installation
   - Added package: `\\DC\Software\Chrome\GoogleChromeStandaloneEnterprise64.msi`

### Troubleshooting
1. Initial permission issues → Verified security settings
2. UNC path error → Recreated package with correct path
3. Final verification: Chrome installed after policy refresh and reboot

## Step 6 - Password Policy Configuration

### Implementation
1. Edited **Default Domain Policy**
2. Navigated to:
   - Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy
3. Configured organization's required password settings

## Troubleshooting

### Common Issues
- **GPOs not applying?**
  - Check Loopback Processing setting
  - Verify correct OU placement
  - Run `gpupdate /force` and reboot
- **Permission errors?**
  - Confirm Domain Computers and Authenticated Users have proper access
  - Check both Share and Security permissions
- **Software not installing?**
  - Validate UNC path accessibility
  - Check Event Viewer for detailed errors

## Conclusion
This lab extension provides hands-on experience with:
- Domain join troubleshooting
- Group Policy creation and management
- Software deployment via GPO
- Enterprise device control policies

## Next Steps
- Explore advanced GPO settings
- Implement login scripts
- Configure folder redirection

## Credits
Guide By **Nicolas Cordischi**

Building on the foundation from Part 1 of our Active Directory Lab series.
