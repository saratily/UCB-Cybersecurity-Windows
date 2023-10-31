# 7-2: Powershell scripting

## Overview
- Use basic PowerShell cmdlets to navigate Windows; manage dir and files.
- Use PowerShell pipelines to retrieve Windows system event logs.
- Combine various shell-scripting concepts such as cmdlets, parameters, piping, conditions, and importing files with data structures.

## issues with cmd
- cmd is easy, short and simple, but can't handle complex operations
- cmd is not consistent 
- e.g. shutdown -s and freedisk -s, -s has different interpretation
- shutdown -d and freedisk -d , -d has different meaning in both
- cmd doesn't support piping

## Powershell cmdlets
- open source - can expand 
- Community support
- can be use to manage Windows server, Windows 10, MS 365 and Azure
- 2 modes: administrator and regular user

### Defensive security
- interact with windows event logs
- extend PowerShell's powerful functionality to enforce cybersecurity policy.

### Offensive security
- it exists on the target’s computer and can be leveraged by attackers.
- used to retrieve a range of information within a network, such as user and server names.
- used to access and maintain access to other networked machines

### CMD
```
@echo off
set size=0
for /r %%x in (System\*) do set /a size+=%%~zx
    echo %size% Bytes
```
### Power shell
The following cmd is running for loop in background
```
> ls C:\Windows | Where-Object {$_.name -like "*system*"}
ls              cmd
C:\Windows      location
|               pipe operator
Where-Object    where is something, just like grep
$_              each item/object
.               accessing object property/column
name            property
-like           key word
"*system*"      wildcard system kwyword for search


- List all the aliases
> Get-Alias ls
OR 
> Alias ls 

CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Alias           ls -> Get-ChildItem

```

### Power shell cmd format
<verb>-<noun>
```
cmdlet          Function                            Equivalent Command
Set-Location    Changes to specified directory              cd
Get-ChildItem   Returns current directory contents          ls, dir
New-Item        Makes new file or directory                 mkdir, touch
Remove-Item     Deletes file or directory                   rm, rmdir
Get-Location    Retrieves path to current directory         pwd
Get-Content     Returns file content cat,                   type
Copy-Item       Copies a file from one location to another  cp
Move-Item       Moves item from one location to another     mv
Write-Output    Prints output                               echo
Get-Alias       Shows aliases from the current session      alias
Get-Help        Retrieves information about command         man
Get-Process     Retrieves processes running on machine      ps
Stop-Process    Stops specific process                      kill
Get-Service     Retrieves list of services              service --status-all
```

### DEMO
```
- Change location to C drive
> Set-Location C:\

- Move reports folder to C: drive
> Move-Item .\reports 'C:\
- If reports folder already exist, the throw error. Use -Force flag to enforce dir move 
> Move-Item .\reports 'C:\ -Force

- List all items in current dir
> Get-ChildItem 

- List all items in a specific dir (Abs path)
> Get-ChildItem C:\

- Create new item (dir or file)
> New-Item  -Path C:\ -Name "Logs" -ItemType Directory

- Remove reports dir
> Remove-Item .\reports

- Get all the commands end with process 
> get-command -name *process

- Get specific command
> get-command Debug-Process

- Launch notepad and open File.txt
> notepad File.txt

- Launch cmd in same terminal
> cmd

- Open and launch cmd in separate window
> start-process -Filepath CMD -Verb Runas

- Close all cmd windows
> stop-process -Name CMD
```

### Activity
1. Since we will eventually decommission the Alex user, move the contracts folder from Alex's desktop directory to C:\.
```
> Move-Item .\contracts 'C:\
```

2. Create Backups and Scripts directories in C:\.
```
> New-Item -Path C:\ -Name "Backups" -ItemType Directory
> New-Item -Path C:\ -Name "Scripts" -ItemType Directory
OR
"Backups", "Scripts" | %{New-Item -Name "$_" -ItemType Directory}
```
3. Also create the C:\Logs directory if you did not do so earlier.
```
> New-Item -Path C:\ -Name "Logs" -ItemType Directory
```

4. Check the contents of the C:\ directory to make sure the Logs, Backups, and Scripts directories exist.
```
> ls C:\
```

5. Bonus - Use Rename-Item to capitalize the contracts directory if it is not already.
```
> Rename-Item C:\contracts C:\contracts1
> Rename-Item C:\contracts1 C:\Contracts
```

## Pipelining - to fetch system event logs
- helps in filtering logs and converting it in a structured format, e..g csv, json, xml, yaml
- Windows env has events happening constantly

### DEMO 
1. Use Get-WinEvent -listlog * to show the list of available logs.
``` Get-WinEvent ```

2. List all logs on the machine,
```
$ Get-WinEvent -LogName *
```
Note: circular - equivalent to rotate

3. Check for and retrieve the names of the system,
```
$ Get-WinEvent -LogName System
```
4. Get latest 10 lines of System event
```
$ Get-WinEvent -LogName System -MaxEvent 10
```
5. Convert to Json format (which will be used by Splunk)
```
$ Get-WinEvent -LogName System -MaxEvent 10 | ConvertTo-Json
```
6. Output to a file (Windows require file extension)
'''
$ Get-WinEvent -LogName System -MaxEvent 10 | ConvertTo-Json  | Out-File -FilePath C:\Logs\RecentSystemLogs.json
'''

7. Get content of file LineNumbers.txt in current dir
```
> Get-Content -Path .\LineNumbers.txt
```

### ACTIVITY - 06_Generating_Windows_Logs

1. Use Get-WinEvent -listlog * to show the list of available logs.
2. Check for and retrieve the names of the security and application logs.
3. For each of the above logs, create a pipeline that:
    - Gets the latest 100 events.
    - Transforms the log to JSON.
4. Outputs the objects in the JSON file to the C:\Logs directory.
Hint: To complete this activity, you will need to use the Get-WinEvent, ConvertTo-Json, and Out-File cmdlets.
5. At the end of this activity, the following files should be in C:\Logs:
- RecentSecurityLogs.json
- RecentApplicationLogs.json

```
 Get-WinEvent -LogName Application -MaxEvent 100 | ConvertTo-Json | Out-File -FilePath C:\Logs\RecentApplicationLogs.json
 

 Get-WinEvent -LogName Security -MaxEvent 100 | ConvertTo-Json | Out-File -FilePath C:\Logs\RecentSecurityLogs.json
 ```
 
## Power shell script
1. Pulls sensitive accounting data and files from a file server to a specified directory.
2. Downloads AppLocker, a program to create rules that allow or block certain executables or scripts from running on a system.
3. Deploys application control policies for AppLocker to restrict executables in a directory so only people in the accounting group can run them.
 
- Can use notepad or VS code to write Power shell script
- variables - $<varname>, eig. $pkgname
- choco - program to manage apps (i.e. get info, unistall, install)

## For loop 
```
foreach ($n in $nums) {
    
}
```

### DEMO
```
notepad .\chocodemo.csv
name, description
itunes, Program
vlc,vlc description
 ```
 
 ```
 $csv=Import-Csv -Path .\chocodemo.csv

foreach ($row in $csv) {
	Write-Output $row
	Write-Output $row.name
}
 ```

 ### ACTIVITY - 09_Removing_Packages
 ```
notepad chocoactivity.csv
 
name,description
mumble,"A voice chat application"
spotify,"An online music player"
telegram,"A cloud-based instant messenger"
skype,"An application for online calling"
teamviewer,"An application for remotely connecting to another computer"
icloud,"Apple's cloud-syncing application"
flashplayeractivex,"An Internet Explorer/Edge add-on that allows one to view Flash animations and videos"
flashplayerppapi,"Chrome/Chromium's implementation of the Adobe Flash Player Plugin"
uplay,"The Ubisoft game launcher"
 ```
 
 ```
 notepad removepackages.ps1

$csv=Import-Csv -Path .\chocoactivity.csv
foreach (e removed!
}$row in $csv) {
	Write-Output $row
	choco uninstall -y $row.name
	Write-Out $row.nam
 ```
