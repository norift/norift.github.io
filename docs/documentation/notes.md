---
layout: default
title: Notes
parent: Documentation
nav_order: 1
---

{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## System:

```
# Get the system disk space
Get-WmiObject -Class Win32_logicaldisk -Filter "DriveType = '3'" | Select-Object -Property DeviceID, DriveType, VolumeName, @{L='FreeSpaceGB';E={"{0:N2}" -f ($_.FreeSpace /1GB)}}, @{L="Capacity";E={"{0:N2}" -f ($_.Size/1GB)}}

# Clear credential manager, ran as user
cmdkey /list | ForEach-Object{if($_ -like "*Target:*" -and $_ -like "*"){cmdkey /del:($_ -replace " ","" -replace "Target:","")}}

# Access user certificates
rundll32.exe cryptui.dll,CryptUIStartCertMgr

# Temporarily disable Symantec Antivirus
start smc -stop
```

## Powershell:

```
# Clone group membership from 1 AD group to another
Add-ADGroupMember -Identity 'newgroup' -Members (Get-ADGroupMember -Identity 'oldgroup' -Recursive)

# Check if special password policy is applied on a account # If no result then the default domain policy applies: Get-ADDefaultDomainPasswordPolicy
Get-ADUserResultantPasswordPolicy $user

# Change domain computername
Rename-Computer –computername OldName –newname NewName –domaincredential Domain\Admin_User –force –restart



```

## Registry:

```
# This registry key when set gives a CLI prompt for credentials instead of a pop-up box!
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\PowerShell\1\ShellIds" -Name ConsolePrompting -Value $true

# Get Windows 10 OS version
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion").ReleaseId

# Disable flash EOL notification message in Internet Explorer, plugin will be removed at the end of next year.
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Main" /v DisableFlashNotificationPolicy /t REG_DWORD /d 1 /f
```

## Installs:

```
# WMIC
wmic product where "name='Tachomaster' and version='02.007.0053'" call uninstall
wmic product where "Vendor like '%Autodesk%'" get Name
wmic product where "Caption like '%Adobe%Flash%'" get Name
```

```
# Get SID's on computer
Reg Query 'HKEY_USERS'

# Get reg detals for apps installed in users profile
Reg Query "HKEY_USERS\S-1-5-21-X-X-X-X\software\microsoft\windows\currentversion\uninstall\"
```

```
# Check who installed software
Get-EventLog -LogName Application -InstanceId 1033 -Message *Adobe* | Format-Table TimeGenerated, UserName, Message -AutoSize -Wrap

# Install report from Event Logs
Get-EventLog -LogName Application -InstanceId 1033 -Newest 1 | Select-Object -ExpandProperty message
Get-WinEvent -FilterHashtable @{logname = ‘setup’} | Format-Table timecreated, message -AutoSize -Wrap
# Other options
Get-WinEvent -FilterHashtable @{logname = ‘Application’; ProviderName='Windows Error Reporting';} | Format-Table timecreated, message -AutoSize -Wrap

# Remote software removal - test
Start-Process  -Wait  -FilePath  "wmic"  -ArgumentList  "product where `"name like '%Adobe Reader%'`" call uninstall" -Hidden

# Remote patch install/removal - test
* How to do .msu's?
wusa.exe C:\Packages\windows10.0-kb4577586-x64_ec16e118cd8b99df185402c7a0c65a31e031a6f0.msu /quiet /norestart
wusa.exe /uninstall /kb:123456 /quiet /norestart
* How to do .cab's?
wusa.exe C:\Packages\spd-x-none_7c1009a5a70cac8d7012a44d2710faf79d7d7fb5.cab /quiet /norestart

# Remote install of HP Drivers
cmd /c 'c:\packages\sp107705.exe' -e -s
cmd /c 'C:\SWSetup\SP107705\setup.exe' -s
```

```
# Silent install of MSODBCSQL
msiexec /i C:\Packages\msodbcsql_17.5.2.1_x64.msi /qn ADDLOCAL=ALL IACCEPTMSODBCSQLLICENSETERMS=YES

# Silent install of SQLNC
msiexec /i C:\Packages\sqlncli.msi /qn ADDLOCAL=ALL IACCEPTSQLNCLILICENSETERMS=YES

# Silent install of VNC Server
cmd /c 'c:\packages\VNC-Server-6.7.2-Windows.exe' /qn REBOOT=ReallySuppress LICENSEKEY=X-X-X-X-X ENABLEAUTOUPDATECHECKS=0 ENABLEANALYTICS=0
** License confirmation: cmd /c 'C:\Program Files\RealVNC\VNC Server\vnclicense.exe' -add X-X-X-X-X
# Silent install of VNC Viewer
cmd /c 'c:\packages\VNC-Viewer-6.20.529-Windows.exe' /qn

# Silent install of Access Runtime 2016
Set-Content 'C:\Packages\office.xml' -value "<Configuration Product=""AccessRT"">`r`n<Display Level=""none"" CompletionNotice=""No"" SuppressModal=""Yes"" NoCancel=""Yes"" AcceptEula=""Yes"" />`r`n<Setting Id=""SETUP_REBOOT"" Value=""Never"" />`r`n</Configuration>"
C:\Packages\AccessRuntime2016_x64_en-us.exe /extract:C:\Packages\AccessRuntime2016_x64_en-us\ /q
C:\Packages\AccessRuntime2016_x64_en-us\setup.exe /config "C:\Packages\office.xml"
Remove-Item "C:\packages\AccessRuntime2016_x64_en-us\" -Recurse -Confirm:$false

# Silent install of NiceLabel-2019
cmd /c 'C:\Packages\NiceLabel2019-BlueYonderVersionONLY.exe' /s LICENSECODE=1234567890 AUTOMATION=FALSE DESIGNER=TRUE RUNTIME=TRUE

# Silent install of Minitab Companion - demo version
cmd /c 'c:\packages\companion5.5.1.0setup.exe' /exenoui /qn ACCEPT_EULA=1 DISABLE_UPDATES=1
- https://www.minitab.com/content/dam/www/en/uploadedfiles/documents/install-guides/MinitabDeploymentGuide_en.pdf

# Silent install of Plantronics Manager
cmd /c 'C:\Packages\PlantronicsHubInstaller.exe' /install /quiet
```


