# 📚 Active Directory - Personal Study Notes
> **My comprehensive AD reference guide for quick revision and practical use**

![Active Directory](https://img.shields.io/badge/Active%20Directory-Windows%20Server-blue?style=for-the-badge&logo=windows)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
![Security](https://img.shields.io/badge/Security-Critical-red?style=for-the-badge&logo=shield)

---

## 📋 Quick Navigation
- [🎯 What is Active Directory?](#-what-is-active-directory)
- [🏗️ Core Components I Need to Know](#️-core-components-i-need-to-know)
- [👥 Managing Users (The Basics)](#-managing-users-the-basics)
- [📁 Organizational Units (OUs) - My Structure](#-organizational-units-ous---my-structure)
- [🎛️ Group Policy Objects (GPOs) - The Game Changer](#️-group-policy-objects-gpos---the-game-changer)
- [🔐 Authentication - How It Really Works](#-authentication---how-it-really-works)
- [💻 PowerShell Commands I Actually Use](#-powershell-commands-i-actually-use)
- [⚠️ Things I Learned the Hard Way](#️-things-i-learned-the-hard-way)
- [🚀 Quick Reference for When I'm Stuck](#-quick-reference-for-when-im-stuck)

---

## 🎯 What is Active Directory?

### The Simple Explanation
**Active Directory = The phone book of your network**

Instead of remembering where every user, computer, and printer is, AD keeps track of everything in one place.

### Why Do I Care?
- **Without AD**: Managing 5 computers = doable, Managing 320 computers = nightmare 😱
- **With AD**: Create user once, works everywhere. Change password policy once, applies everywhere.

### Real-World Analogy
Think of your school/work login:
- Same username/password works on ANY computer on campus
- Admin can block you from Control Panel on ALL computers
- That's Active Directory doing its magic! ✨

---

## 🏗️ Core Components I Need to Know

### 🏢 Active Directory Domain Service (AD DS)
> **Think of it as:** The master catalog of everything

**What it stores:**
```
📁 Users (People + Service accounts)
💻 Computers (Workstations, Servers, Domain Controllers)
👥 Groups (Who can access what)
🖨️ Resources (Printers, shared folders, etc.)
```

**Key Point:** This is the heart of everything. If this breaks, everything breaks.

---

### 👤 Security Principals
> **Translation:** Things that can log in and do stuff

#### Types I'll Deal With:

**1. User Accounts**
- **People**: Regular employees (john.doe@company.com)
- **Service Accounts**: For applications (svc_sql, svc_web)

**2. Computer Accounts**
- **Format**: COMPUTERNAME$ (e.g., LAPTOP01$)
- **Password**: 120 random characters (changes automatically)
- **Important**: Never try to use these manually!

**3. Groups**
- **Security Groups**: Control access to stuff
- **Distribution Groups**: Just for email lists

---

### 🏢 Default Groups I Should Know

| Group Name | What They Can Do | When I'd Use This |
|------------|------------------|-------------------|
| **Domain Admins** | Everything everywhere | Full control (be careful!) |
| **Server Operators** | Manage servers, not users | For server maintenance folks |
| **Backup Operators** | Access any file for backup | Backup software accounts |
| **Account Operators** | Create/modify users | Help desk team |
| **Domain Users** | All users automatically | Default permissions |

> **🚨 Remember:** Domain Admins = Nuclear option. Use sparingly!

---

## 👥 Managing Users (The Basics)

### 🛠️ Active Directory Users and Computers (ADUC)
> **How to get there:** Start Menu → "Active Directory Users and Computers"

#### What I'll See:
```
📁 thm.local (my domain)
├── 📁 Builtin (Windows default groups)
├── 📁 Computers (New computers go here by default)
├── 📁 Domain Controllers (The important servers)
├── 📁 Users (Default users and groups)
└── 📁 My Custom OUs (Departments, etc.)
```

#### Cool Tricks I Learned:
- **Right-click + "Find"**: Search for anything
- **F5**: Refresh (when things don't show up)
- **Advanced Features**: View → Advanced Features (shows more stuff)

---

### 🔑 Password Management

#### The PowerShell Way (Faster):
```powershell
# Reset someone's password
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose

# Make them change it next login
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

#### Why This Matters:
- Help desk can reset passwords without full admin rights
- Users can't keep using passwords I set
- Everything is logged and traceable

---

### 🎭 Delegation (Give Others Limited Power)

#### What I Learned:
Instead of making everyone a Domain Admin, I can give specific people specific powers.

#### Example - Let IT Support Reset Passwords:
1. Right-click the OU (e.g., Sales)
2. "Delegate Control"
3. Add user (e.g., phillip)
4. Choose "Reset user passwords and force password change at next logon"
5. Done! Phillip can now reset Sales passwords but nothing else

#### Real-World Use:
- Help desk: Password resets only
- Department managers: Manage their own people
- Regional IT: Manage their region only

---

## 📁 Organizational Units (OUs) - My Structure

### 🤔 What Are OUs?
> **Think of them as:** Folders for organizing users and computers

### 📐 How I Should Structure Them:

#### Good Structure (Mirrors Business):
```
🏢 company.local
├── 💻 Workstations
│   ├── Laptops
│   └── Desktops
├── 🖥️ Servers
│   ├── Web Servers
│   └── Database Servers
├── 👥 Users
│   ├── IT Department
│   ├── Sales Department
│   ├── Marketing Department
│   └── Management
└── 🔧 Domain Controllers (Windows creates this)
```

#### Why This Structure Works:
- **IT policies** → Different for servers vs laptops
- **User policies** → Different for departments
- **Security** → Each OU can have different permissions

---

### 🗑️ Deleting OUs (The Gotcha)

#### The Problem:
Right-click → Delete = "Cannot delete, object is protected"

#### The Solution:
1. **View** → **Advanced Features** (enable this first!)
2. Right-click OU → **Properties**
3. **Object** tab → Uncheck **"Protect object from accidental deletion"**
4. **Apply** → **OK**
5. Now I can delete it

> **💡 Pro Tip:** This protection exists for a reason. Double-check before deleting!

---

### 🎯 OUs vs Groups - When to Use What?

| Need | Use | Why |
|------|-----|-----|
| Apply different computer policies | **OUs** | Policies apply to OUs |
| Give access to shared folder | **Groups** | Permissions go to groups |
| Organize by department | **OUs** | Structure and policies |
| Give access to printer | **Groups** | Resource permissions |

> **Key Rule:** User can be in ONE OU, but MANY groups

---

## 🎛️ Group Policy Objects (GPOs) - The Game Changer

### 🤯 What GPOs Actually Do
> **Translation:** Set rules for computers and users across the network

#### The Magic:
- Create rule once → Applies to hundreds of computers
- Want to block Control Panel? → One GPO to rule them all
- Need auto-lock screens? → Set it and forget it

---

### 🎮 Group Policy Management Console (GPMC)
> **How to get there:** Start Menu → "Group Policy Management"

#### What I'll See:
```
📁 Forest: company.local
└── 📁 Domains
    └── 📁 company.local
        ├── 📋 Default Domain Policy (applies to everyone)
        ├── 📁 Domain Controllers
        │   └── 📋 Default Domain Controllers Policy
        ├── 📁 Group Policy Objects (my GPOs live here)
        └── 📁 My OUs (where I link GPOs)
```

---

### 🏗️ How GPOs Work

#### The Process:
1. **Create** GPO in "Group Policy Objects"
2. **Configure** settings in the GPO
3. **Link** GPO to OU where I want it to apply
4. **Wait** (or run `gpupdate /force`)

#### Important Concepts:
- **Computer Configuration**: Applies to machines
- **User Configuration**: Applies to people
- **Inheritance**: Child OUs get parent policies too
- **Security Filtering**: Can target specific users/groups

---

### 🛠️ Common GPOs I Actually Use

#### 1. Block Control Panel Access
> **Goal:** Stop non-IT users from changing system settings

**Path:** `User Configuration → Administrative Templates → Control Panel`
**Setting:** `Prohibit access to Control Panel and PC settings: Enabled`
**Link to:** Marketing, Sales, Management OUs (NOT IT!)

```powershell
# Force update to test immediately
gpupdate /force
```

---

#### 2. Auto-Lock Screen After 5 Minutes
> **Goal:** Secure unattended workstations

**Path:** `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options`
**Setting:** `Interactive logon: Machine inactivity limit: 300 seconds`
**Link to:** Root domain (applies to all computers)

---

#### 3. Password Policy
> **Goal:** Enforce strong passwords

**Path:** `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`
**Common Settings:**
- Minimum password length: 10 characters
- Password complexity: Enabled
- Maximum password age: 90 days

---

### 🔄 GPO Distribution & Troubleshooting

#### How GPOs Spread:
- **SYSVOL Share**: `\\domain.com\SYSVOL`
- **Location**: `C:\Windows\SYSVOL\sysvol\` on DCs
- **Update Frequency**: Every 90 minutes (computers), 90 minutes (users)

#### When GPOs Don't Work:
```powershell
# Force immediate update
gpupdate /force

# See what policies are applied
gpresult /r

# Detailed report to HTML file
gpresult /h gpo_report.html /f
```

---

## 🔐 Authentication - How It Really Works

### 🎫 Kerberos (The Modern Way)
> **Used by:** All recent Windows versions (default)

#### The Ticket System:
Think of it like a movie theater:
1. **Buy ticket at box office** (get TGT from Domain Controller)
2. **Show ticket to enter theater** (use TGT to get service tickets)
3. **Show ticket to usher for specific movie** (use service ticket to access resource)

#### The Technical Flow:
```
1. User → Domain Controller: "I'm john.doe, here's my encrypted timestamp"
2. DC → User: "Here's your Ticket Granting Ticket (TGT) + Session Key"
3. User → DC: "I want to access FileServer01, here's my TGT"
4. DC → User: "Here's your service ticket for FileServer01"
5. User → FileServer01: "Here's my service ticket"
6. FileServer01: "Welcome! You're authenticated"
```

#### Why It's Better:
- **Single Sign-On**: Get ticket once, use everywhere
- **No passwords on network**: Only encrypted tickets travel
- **Efficient**: Don't need to contact DC for every service

---

### 🔄 NetNTLM (The Legacy Way)
> **Used by:** Older systems, some applications

#### The Challenge-Response:
```
1. Client → Server: "I want to connect"
2. Server → Client: "Prove it. Here's a random challenge"
3. Client → Server: "Here's my response (using my password hash)"
4. Server → DC: "Is this response correct for this challenge?"
5. DC → Server: "Yes/No"
6. Server → Client: "Welcome/Go away"
```

#### Why It's Problematic:
- More network traffic
- Vulnerable to certain attacks
- No single sign-on benefits

> **🎯 My Takeaway:** Kerberos = good, NetNTLM = avoid when possible

---

## 💻 PowerShell Commands I Actually Use

### 🚀 Getting Started
```powershell
# Load AD module (always do this first)
Import-Module ActiveDirectory

# Get help for any AD command
Get-Help Get-ADUser -Examples
```

---

### 👤 User Management

#### Create New User:
```powershell
New-ADUser -Name "Jane Smith" `
           -SamAccountName "jane.smith" `
           -UserPrincipalName "jane.smith@company.com" `
           -Path "OU=IT,OU=Users,DC=company,DC=com" `
           -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
           -Enabled $true
```

#### Get User Info:
```powershell
# Basic info
Get-ADUser -Identity "john.doe"

# Everything
Get-ADUser -Identity "john.doe" -Properties *

# Specific properties
Get-ADUser -Identity "john.doe" -Properties LastLogonDate, Department
```

#### Modify User:
```powershell
# Change department
Set-ADUser -Identity "john.doe" -Department "IT"

# Disable account
Disable-ADAccount -Identity "john.doe"

# Enable account
Enable-ADAccount -Identity "john.doe"
```

---

### 👥 Group Management

#### Add User to Group:
```powershell
Add-ADGroupMember -Identity "IT Admins" -Members "john.doe"
```

#### Remove User from Group:
```powershell
Remove-ADGroupMember -Identity "IT Admins" -Members "john.doe"
```

#### See Group Members:
```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

---

### 💻 Computer Management

#### Find Computers:
```powershell
# All computers in specific OU
Get-ADComputer -SearchBase "OU=Workstations,DC=company,DC=com" -Filter *

# Computers not logged in for 90 days
$Date = (Get-Date).AddDays(-90)
Get-ADComputer -Filter {LastLogonDate -lt $Date} -Properties LastLogonDate
```

---

### 📊 Reporting Queries I Use

#### Security Auditing:
```powershell
# Find all disabled users
Get-ADUser -Filter {Enabled -eq $false} | Select Name, LastLogonDate

# Find users with passwords that never expire
Get-ADUser -Filter {PasswordNeverExpires -eq $true} -Properties PasswordNeverExpires

# Find empty groups
Get-ADGroup -Filter * | Where-Object {-not (Get-ADGroupMember $_)} | Select Name

# Domain Admins report
Get-ADGroupMember -Identity "Domain Admins" | Get-ADUser -Properties LastLogonDate, Department
```

#### Bulk Operations:
```powershell
# Disable all users in specific OU
Get-ADUser -SearchBase "OU=Temp,DC=company,DC=com" -Filter * | Disable-ADAccount

# Export user list to CSV
Get-ADUser -Filter * -Properties Department, LastLogonDate | 
Export-Csv -Path "C:\Reports\AllUsers.csv" -NoTypeInformation
```

---

### 🎯 My PowerShell Pro Tips

#### Always Use -WhatIf First:
```powershell
# See what would happen without doing it
Get-ADUser -Filter {Department -eq "Sales"} | Set-ADUser -Department "Marketing" -WhatIf
```

#### Pipe Commands for Efficiency:
```powershell
# Find and disable old computer accounts in one line
Get-ADComputer -Filter {LastLogonDate -lt (Get-Date).AddDays(-180)} | Disable-ADAccount
```

#### Save Frequent Commands:
```powershell
# Create function for common tasks
function Get-StaleComputers {
    param([int]$Days = 90)
    $Date = (Get-Date).AddDays(-$Days)
    Get-ADComputer -Filter {LastLogonDate -lt $Date} -Properties LastLogonDate
}

# Use it
Get-StaleComputers -Days 120
```

---

## ⚠️ Things I Learned the Hard Way

### 🚨 Critical Mistakes to Avoid

#### 1. "Oops, I Deleted the Wrong OU"
**Problem:** Deleted OU with 50 users inside
**Solution:** 
- Always check contents first: `Get-ADUser -SearchBase "OU=ToDelete,DC=company,DC=com" -Filter *`
- Move users out before deleting OU
- Test in lab environment first

#### 2. "Why Isn't My GPO Working?"
**Common Issues:**
- GPO created but not linked to OU
- Wrong OU targeted
- Security filtering blocking users
- Inheritance blocked
- Time sync issues

**Debug Steps:**
```powershell
# On target machine
gpupdate /force
gpresult /r
# Check Event Viewer → Windows Logs → System
```

#### 3. "I Broke Domain Authentication"
**What Happened:** Modified Default Domain Policy incorrectly
**Lesson:** Never edit Default Domain Policy unless you know exactly what you're doing
**Better Approach:** Create new GPO, link it, test it, then consider modifying defaults

---

### 🔧 Recovery Procedures

#### When Things Go Wrong:
1. **Don't Panic** - Most things are recoverable
2. **Check Recent Changes** - What did I change recently?
3. **Use Event Viewer** - Errors are usually logged
4. **Test on Single Machine** - Before rolling back domain-wide changes
5. **Have Rollback Plan** - Always know how to undo changes

#### Emergency Contacts:
- Keep list of all Domain Controllers
- Know how to start in Safe Mode
- Understand authoritative restore process
- Keep AD backup schedule documented

---

## 🚀 Quick Reference for When I'm Stuck

### 🔍 Common MMC Snap-ins
| Tool | Command | What It Does |
|------|---------|--------------|
| AD Users & Computers | `dsa.msc` | Manage users, groups, OUs |
| Group Policy Management | `gpmc.msc` | Manage GPOs |
| DNS Manager | `dnsmgmt.msc` | Manage DNS zones |
| Event Viewer | `eventvwr.msc` | Check logs |
| Services | `services.msc` | Manage Windows services |

### ⚡ Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Win + R` | Run dialog |
| `F5` | Refresh current view |
| `Ctrl + F` | Find/Search |
| `Alt + Enter` | Properties |
| `Ctrl + Shift + Enter` | Run as administrator |

### 🆘 Emergency Commands
```powershell
# Check domain controller health
dcdiag /v

# Test secure channel
Test-ComputerSecureChannel -Verbose

# Force AD replication
repadmin /syncall /AdeP

# Check FSMO roles
netdom query fsmo

# Force GPO update
gpupdate /force

# Check time sync (critical for Kerberos)
w32tm /query /status
```

### 📊 One-Liner Reports
```powershell
# Quick user count by OU
Get-ADUser -Filter * -Properties DistinguishedName | Group-Object {($_.DistinguishedName -split ',')[1]} | Select Name, Count

# Disabled accounts
Get-ADUser -Filter {Enabled -eq $false} | Measure-Object | Select Count

# Computers by OS
Get-ADComputer -Filter * -Properties OperatingSystem | Group-Object OperatingSystem | Select Name, Count

# Groups with no members
Get-ADGroup -Filter * | Where-Object {-not (Get-ADGroupMember $_)} | Select Name
```

---

## 📚 Study Reminders for Future Me

### 🎯 Before Making Changes:
- [ ] Do I have a rollback plan?
- [ ] Have I tested this in lab?
- [ ] Am I in a maintenance window?
- [ ] Are backups current?
- [ ] Have I documented what I'm doing?

### 🔄 Regular Maintenance Tasks:
- [ ] Review Domain Admin membership monthly
- [ ] Audit disabled accounts quarterly
- [ ] Check for stale computer accounts
- [ ] Review GPO settings annually
- [ ] Test backup/restore procedures
- [ ] Monitor replication health
- [ ] Review delegation permissions

### 📖 Concepts to Review Regularly:
- Trust relationships (direction vs access)
- GPO processing order and inheritance
- Kerberos ticket flow
- FSMO roles and placement
- DNS integration with AD
- Time synchronization importance

---

## 🎉 Final Thoughts

Active Directory is like a city's infrastructure - when it works well, nobody notices. When it breaks, everything stops. The key is understanding the fundamentals and building good habits:

1. **Plan before you implement**
2. **Test before you deploy**
3. **Document what you do**
4. **Monitor what you've built**
5. **Have a recovery plan**

Remember: Every domain admin was once confused by OUs vs Groups. Every expert has accidentally deleted something important. The difference between a junior and senior admin isn't that seniors don't make mistakes - it's that they make fewer of them and recover faster when they do.

---

*These notes are my personal reference guide based on hands-on learning and real-world experience. They're meant to be practical, not exhaustive.*

**Last Updated:** June 2025  
**Source:** TryHackMe Active Directory Basics Lab + Personal Experience  
**Status:** Living document - updated as I learn more

---

### 🔗 Additional Resources to Bookmark:
- [Microsoft AD Documentation](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [PowerShell AD Module Reference](https://docs.microsoft.com/en-us/powershell/module/activedirectory/)
- [Group Policy Reference](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791(v=ws.11))

---

**⭐ Star this repo if these notes helped you!**
