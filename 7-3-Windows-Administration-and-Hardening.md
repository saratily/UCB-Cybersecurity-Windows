# 7.3 Active Directory DOmain Service (AD)

## Overview

- Hw AD works - to manage enterprise-scale environments.
- Define domain controllers as servers that manage AD authentication and authorization.
- Use AD -  to create organizational units (OU), users, and groups.
- Create & link Group Policy Objects (GPO) to user

## Active Directory is the central

- db to manage system for enterprise-scale Windows environments
- used within organization for
    |_ Security analysis
    |_Thread hunters
    |_Forensics experts
    |_Incidence repsonders
- used by 
    |_Penetration testing experts
    |_Thread intelligence experts
    |_Malware recerse eng experts

- If org grows 20 -> 100, ensure everyone has same access 

```structure
- Objects (users, groups, computers, fiels, printers, etc.)
    |_ Security principals  - AD obj to authenticated users & groups, e.g. require pwd
                            - define permissions to them access

    |_ Resources    - requires authorization
                    - e.g files, networking components, and printers
                    - users need permission to access. 
                    - Permissions depend on roles and responsibilities

```

- Remember the principle of least privilege from the Linux SysAdmin units?
-- Donot give full acess to everyone

## AD Architecture

## AD AUthentication

### LDAP - Lightweight Directory Access Protocol

- protocol managing (add, del, and edit obj) 
. AD -> journal, LDAP -> pencil and eraser

### Kerberos Overview

A ticket-based

- authentication protocol,
- default on Windows Server domain
- provides direct encrypted sessions b/w users and networked.
- 7 step process:
- Key Distribution Center (KDC) - db of credentials, Auth Server & Ticket Granting Server.
-  Ticket Granting Ticket (TGT) - cached and permits Bob to request more tickets, 
-  Ticket Granting Server (TGS)
1. -> Bob�s req  KDC seeking a
2. <- After verified, Bob receives a TGT -> Bob can request access to resources.
3. -> Req access to file server -> send TGT to TGS 
4. <- KDC validate TGT -> send BOB encrypted service ticket, and session key 
5. -> Bob use session key to encrypt msg + service ticket -> file server
6. file server uses session key to decrypt service ticket 
7. <- file server -> encrypted new msg -> bob -> info about file server 
8. Bob decrypts msg

### NTLM - New Technology LAN Manager
- An authentication protocol
- Outdated bcz pass hash attacks



## SETUP
### ACTIVITY - 00_Windows_Lab
Windows RDP Host machine: u(azadmin) p: (p4ssw0rd*)
    |_ Hyper V Manager
        |_ Windows server: u: (sysadmin) p: (p4ssw0rd*)
        |_ Windows 10: u: (sysadmin) p: (cybersecurity)

This Windows 10 and Windows Server virtual machine will require an extension of the evaluation licenses so that they do not shut down abruptly.
```
do the following for both windows server and Windows 10
cmd ->  Run as administrator -> slmgr.vbs /rearm  msg "Command completed successfully-> restart VM
```

## Creating OU, Users, and Groups

Organizational units (OUs) - logical group of assets and accounts.
Groups: Should be unique across all the OU (however, OU and group name can match)
        rename
        drag and drop
        copy 

### DEMO
0. Search for Active Directory Users and Computers
```
windows search -> Active Directory Users and Computers
GOODCORP.NET              -> main organization
```
1. Create a new domain organizational unit called GC Users.
```
GOODCORP.NET ->
    right click -> New -> 
        Organization unit ->
            GC Users 
                uncheck 'protect container form accidental deletion'  -> 
            Hit OK

```
2. Create a sub-OU called Marketing.
```
Right click GC Users (OU)  -> New -> Organization unit -> marketing
uncheck "protect container form accidental deletion"
```
3. Create a user, Caroline, under the GC Users > Marketing OU.
```
Right click marketing -> New -> User -> Caroline
First name: Caroline
Full name: Caroline
User Logon name: Caroline
User logon name (pre Windows 2000): GOODCORP\caroline
-> Next

Password: 
Confirm Password:
Check: User must change password on next logon
-> Next

Full name: Caroline
User logon name: Caroline@GOODCORP.NET
The user must change the password at next logon.
-> Finish

```
4. Create a group, Marketing, under the GC Users > Marketing OU.
```
Right click marketing(OU) -> New -> Group -> Marketing
Group Scope
    |_ Domain Local
    |_ Global (checked)
    |_ Universal
Group Type
    |_ Security (checked)
    |_ Distribution 
-> OK
```
5. Add Caroline to marketing group
```
Right click on Caroline ->
Add to a group ->
Manually enter correct group name -> Marketing (case insensetive MARketng will work as well)
Check name or enter-> if manually entered name is underlined, that mean group exist ->
OK
---------------------------------------------------------------------------------
To verify: 
Right click on Marketing Group -> 
Properties ->
Members tab ->
Caroline user is listed
OR
Right Click on Caroline (User) -> 
Properties -> 
Member Of -> 
Marketing group is listed
```

### ACTIVITY - 04_Creating_OUs_Users_Groups (Same as demo)

1. Create a top-level GC Users organizational unit for each department's users.
2. Create a Sales sub-OU.
3. Create a Development sub-OU.
4. Create a user, Bob, in the Sales OU. Set their password to Ilovesales!.
5. Create a group, Sales, within the Sales OU.
6. Add user, Bob, to the group Sales.
7. Create a user, Andrew, in the Development OU. Set their password to Ilovedev!.
8. Create a group, Development, within the Development OU.
9. Add user, Andrew, to the group Development.
10. Move the Windows 10 machine in the existing default Computers directory to a new GC Computers OU.
11. Create a GC Computers OU. It should be on the same level as GC Users.

## Group Policy Object (GPO)

- set of policy settings that contain one or more group policy.
e.g.
Implement password complexity requirements for accountants.
also
Deploy some form of anti-malware software when they next log on

### DEMO

0. Search for Group Policy Management

```structure
Windows Search ->  Group Policy Management
Forest      -> top level 
    |_ Domians
            |_ GOODCORP.NET         -> main organization
                |_ Group policy Object

```

1. Create a Group Policy Object. (e.g. No Control Panel )

```text
Right click on Group Policy Object -> new -> 
Enter policy name -> No Control Panel -> OK
```

2. Edit the individual policies for our Group Policy Object.

```structure
Right Click on GPO (No Control Panel) -> Edit
User Configuration
    |_ Plicies
        |_ Administration templates             -> pre made GPO, that can be used
            |_ Control Panel 
                |_ Prohibition access to control panel and PC settings
                    double click

Select Enable -> 
Optional: add comment -> OK
```

3. Link the Group Policy Object to an organizational unit. (No Control Panel  -> Sales)

```structure
Forest      -> top level 
    |_ Domians
            |_ GOODCORP.NET         -> main organization
                |_GC Users
                    |_ Development
                    |_ MArketing
                    |_ Sales

Right click sales -> Link to Exisitng GPO -> select No Control Panel from textarea -> OK
```

4. Within the Windows 10 machine, retrieve the latest Active Directory changes.

```text
NOte: Instructions say login as sysAdmin, then do gpupdate. However, it worked as Bob
C:\Users\Bob > gpupdate
Updating Policy ...

Computer policy update has completed successfully
User policy update has completed successfully

```

5. Verify if the GPO worked. (Bob belongs to sales)

```text
Login as Bob -> 
Windows Search -> Control Panel -> Run

Restrictions
This operation has been cancelled due to restrictons in effect on this computer. Please contact your system administration
```

### ACTIVITY - 

1. Create the GPO. 
2. Name the GPO `Limit Settings`.
3. Right-click the GPO to Edit... its policies.

```structure
Right Click on GPO (No Control Panel) -> Edit
Computer Configuration
    |_ Policies
        |_ Administration templates             -> pre made GPO, that can be used
            |_ Control Panel 
                |_ Setting Page Visibility

Enabled -> Under Settings Page Visibility: showonly:about;themes -> Apply -> OK
```

4. After you have enabled the policy, link the GPO to the Sales OU.

--------------------------------------------------------------------------------

5. Add the Sales group to the Remote Desktop Users group like shown earlier, if you haven't already.

```structure
Windows Search -> Active Directory Users and Computers
GOODCORP.NET
    |_GC Users
        |_ Development
        |_ Marketing
        |_ Sales

Right click Sales -> Add to a group -> Enter Remote -> click Check Names.

Select Remote Desktop Users -> OK -> OK
```

6. Refresh these policy and group changes on the Windows 10 machine.

```command
Windows 10 -> login as sysadmin
Windows Search -> cmd -> Run as administrator
C:\Users\sysadmin > gpupdate
Updating Policy ...

Computer policy update has completed successfully
User policy update has completed successfully
```

- Go back to the Windows 10 machine, open a CMD or PowerShell window and enter:
- This will update the Windows 10 machine with your latest changes.
- Take a quick moment to look at the default Windows Settings.
- Click the Start menu. Then click the cogwheel icon to launch the Settings menu.
- You don't have to do anything here, but notice the different settings your sysadmin account has access to.

7. Test the Group Policy changes as Bob. Verify that Bob's account has limited access to Windows Settings by doing the following:

- Log into the Windows 10 machine as the local sysadmin account and run gpupdate to pull the latest updates.
- Toggle Enhanced Session Mode OFF then log into the Windows 10 machine as Bob with password Ilovesales!.
- Once you're back in the Windows 10 machine, click the Windows button in the bottom left and then click the Settings icon.
- If you created the GPO properly, we should only see two entries there: System and Personalization.
- This indicates that the Group Policy Object was successfully applied. The standard Windows Settings screen has many more options.
- Know that system administrators will often restrict access to various operating system settings to prevent users from making irreversible or potentially dangerous changes to their system.

8. Authenticate back to our local sysadmin account:

- Log out of Bob's account by toggling Enhanced Session Mode ON again.
- Back at the Windows lock screen, click the screen and select Other user.
- For the username, enter .\sysadmin and cybersecurity for the password. You'll notice that the Sign in to: selection at the bottom automatically changes when you type .\.
- By toggling the session mode, you can use GoodCorp.net\Bob to sign into Bob's domain account, and .\sysadmin to sign into the local sysadmin account.