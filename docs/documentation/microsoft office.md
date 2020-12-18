---
layout: default
title: Microsoft Office
parent: Documentation
nav_order: 2
---

{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

```
14.0 = Office 2010
15.0 = Office 2013
16.0 = Office 2016
```

## Office Installation:
### "Error 1907. Could not register font. Verify that you have sufficient permissions to install fonts, and that the system supports this font."
This error is fixed by running "SFC /SCANNOW" which will resolve a file system issue, the command below is intended for a remote powershell session.
```
Start-Process -FilePath "${env:Windir}\System32\SFC.EXE" -ArgumentList '/scannow' -Wait -Verb RunAs -WindowStyle hidden
```

## Outlook:
### App v16.0.4266.1001 crashing with emails that have attachments
*Microsoft Outlook 2016 may crash when using the Symantec Endpoint Protection (SEP) Outlook Scanner Add-in. Uninstalling or disabling the Symantec add-in resolves the symptoms.*

https://knowledge.broadcom.com/external/article/170414/outlook-2016-crashes-when-using-the-endp.html
- [Office 2013: KB4011282 Download](https://download.microsoft.com/download/6/E/8/6E8FE290-E440-40A4-8F99-5D73CCEF0459/outlook2013-kb4011282-fullfile-x64-glb.exe)
- [Office 2016: KB4011570 Download](https://download.microsoft.com/download/4/B/D/4BDAB50A-A117-45A1-B104-31D946326485/outlook2016-kb4011570-fullfile-x64-glb.exe)

```
Faulting application name: OUTLOOK.EXE, version: 16.0.4266.1001, time stamp: 0x55ba1876
Faulting module name: OUTLOOK.EXE, version: 16.0.4266.1001, time stamp: 0x55ba1876
Exception code: 0xc0000005
Fault offset: 0x000000000002a10f
Faulting process id: 0x26c0
Faulting application start time: 0x01d6ad0dbb0ee84c
Faulting application path: <C:\Program Files\Microsoft Office\Office16\OUTLOOK.EXE>
Faulting module path: <C:\Program Files\Microsoft Office\Office16\OUTLOOK.EXE>
Report Id: 912141fa-54e5-430c-b8dd-adb8a914bfa7
```
### Email rule - run a script
This feature was by default hidden after a security update, it can be enabled through the users registry.
```
reg add "HKEY_CURRENT_USER\Software\Microsoft\Office\XX.X\Outlook\Security" /v EnableUnsafeClientMailRules /t REG_DWORD /d 1 /f
```
### "The Delegates settings were not saved correctly. Unable to activate send-on-behalf-of list. You do not have sufficient permission to perform this operation on this object."
*This error can occur when the Delegates list contains a mailbox user who no longer exists in the organization. To fix the error remove the non-existent user from the Delegates list before you attempt to add other Delegates or change the Delegates settings.*

The first command is to check the attribute, the next following 2 are either to clear a specific inactive user or to remove them all.
```
Get-ADUser $user -Properties publicDelegates | Select-Object -ExpandProperty publicDelegates

Set-ADUser -Remove @{PublicDelegates="UID"}
Set-ADUser -clear PublicDelegates
```


## Excel:
### App v16.0.4978.1000 crashing with error reference to chart.dll
The issue is caused by KB4018319 and is solved by installing the appropriate patch:

- [Microsoft Office 2013 - KB2986229 Download](https://download.microsoft.com/download/B/9/9/B99D8FC5-569A-42F0-B0AD-089BB246C327/oart2013-kb2986229-fullfile-x64-glb.exe)
- [Microsoft Office 2016 - KB4011128 Download](https://download.microsoft.com/download/1/A/6/1A69A731-CEC8-4D3F-BA81-0E801E8B0C7D/chart2016-kb4011128-fullfile-x64-glb.exe)

```
Get-EventLog -LogName Application -InstanceId 1000 -Message *EXCEL.exe* | Select-Object -ExpandProperty message
```
```
Faulting application name: EXCEL.EXE, version: 16.0.4978.1000, time stamp: 0x5e451d6b
Faulting module name: chart.dll, version: 16.0.4678.1000, time stamp: 0x5aa7ed63
Exception code: 0xc0000005
Fault offset: 0x00000000001ba0ac
Faulting process id: 0x87c
Faulting application start time: 0x01d6bc44e1f3dcfd
Faulting application path: C:\Program Files\Microsoft Office\Office16\EXCEL.EXE
Faulting module path: C:\Program Files\Microsoft Office\Office16\chart.dll
Report Id: 91239ca4-9421-40c5-9383-ee093ae9cf0e
```






