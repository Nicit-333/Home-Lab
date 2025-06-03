# Home Lab Part 2: Domain Login & Group Policy Management

## 🛠️ Issue 1: Domain Login Failure on Client1

### Problem Summary
- **Client1** (Windows 10, domain-joined) failed to authenticate domain users
- Client1 was missing from the **"Computers"** container in Active Directory (AD)

### 🔍 Troubleshooting Steps
1. Verified user account in AD → Credentials and domain format were correct
2. Checked AD "Computers" container → Client1 was not listed, suggesting improper domain join
3. On Client1 login screen:
   - Selected "Other user" → Domain name appeared correctly
   - Logged in via local administrator account for access

### 🔧 Resolution
1. Disjoined Client1 from the domain:
   - Settings > System > About > Rename this PC (advanced)
   - Changed to workgroup, restarted
2. Rejoined Client1 to the domain:
   - Repeated steps, entered domain: mydomain.com
   - Provided domain admin credentials, restarted
3. Confirmed in AD: Client1 now appeared under "Computers"
4. Tested domain login → Success

### ✅ Root Cause
- Likely an incomplete/corrupted domain join or broken secure channel between Client1 and the DC

---

## 🛠️ Group Policy Deployment

### Step 1: Setting Up Group Policy Management (GPMC)
- Verified GPMC installed via:
  - Server Manager > Add Roles and Features > Features > Group Policy Management

### Creating an Organizational Unit (OU)
- Opened GPMC, right-clicked domain → New OU → Named "TestComputers"
- Moved Client1 from AD "Computers" to "TestComputers" OU

---

### Step 2: Deploying Wallpaper via GPO

#### Steps:
1. Created shared folder:
   - Path: `C:\Wallpapers` → Saved wallpaper image (`ilovehousemusic.jpg`)
2. Shared folder permissions:
   - Advanced Sharing > Share name: Wallpapers
   - Removed "Everyone", added Domain Computers (Read access)
   - UNC Path: `\\DC\Wallpapers\ilovehousemusic.jpg`
   - Issue: Client1 couldn't access → Fixed by adding Authenticated Users to permissions
3. Created GPO:
   - OU: TestComputers → New GPO → Named "Set Desktop Wallpaper"
   - Configured:
     - User Configuration > Policies > Admin Templates > Desktop > Desktop Wallpaper
     - Enabled, set UNC path, wallpaper style: Fill
4. Applied GPO:
   - Ran `gpupdate /force` on Client1 → No effect
   - Fix: Enabled Loopback Processing (Merge mode) under:
     - Computer Configuration > Policies > Admin Templates > System > Group Policy
   - Result: Wallpaper applied after reboot

---

### Step 3: Disabling USB Storage (Allow Other HID Devices)
- **GPO Name**: "Disable USB Storage" (Linked to TestComputers OU)
- **Configured**:
  - Computer Configuration > Policies > Admin Templates > System > Removable Storage Access
  - Enabled:
    - Deny read access
    - Deny write access
- **Tested**:
  - USB drives blocked; mice/keyboards (HID class) worked

### Enforcing Password Policies
- **Note**: Password policies are domain-wide (set in Default Domain Policy)
- **Steps**:
  - Edited Default Domain Policy → Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy

---

## 🛠️ Step 4: Deploying Chrome via GPO

### Steps:
1. Downloaded Chrome MSI:
   - From: [Chrome Enterprise Download](https://chromeenterprise.google/browser/download/) (64-bit MSI)
   - Issue: Page wouldn't load → Fixed by adding 8.8.8.8 as alternate DNS
2. Created shared folder:
   - Path: `C:\Software\Chrome`
   - Shared as `\\DC\Software` with Read access for:
     - Domain Computers
     - Authenticated Users
3. Created GPO:
   - OU: TestComputers → New GPO → Named "Deploy Chrome"
   - Configured:
     - Computer Configuration > Policies > Software Settings > Software Installation
     - Added package via UNC Path: `\\DC\Software\Chrome\GoogleChromeStandaloneEnterprise64.msi`
4. Troubleshooting:
   - Issue 1: Client1 couldn't access UNC → Fixed security permissions (added Authenticated Users)
   - Issue 2: Wrong UNC path in GPO → Recreated package with correct path
5. Applied GPO:
   - Ran `gpupdate /force` + reboot → Chrome installed successfully

---

### ✅ Key Takeaways
- **Domain join issues** → Rejoin domain + verify AD objects
- **GPO deployment** requires:
  - Correct permissions (Domain Computers/Authenticated Users)
  - Accurate UNC paths
  - Loopback Processing for user policies on computers
- **Software deployment** needs shared folder + MSI installer
