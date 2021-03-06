---
layout: default
title: Microsoft Office
parent: Documentation
nav_order: 4
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

Spreadsheet Compare is only available with Office Professional Plus 2013 or Microsoft 365 Apps for enterprise
Excel: Power Pivot - Requires: Office 2013 professional plus and PowerBI Desktop
Office 365 F3 license - allows web access to Yammer and OneDrive via the browser


get-childitem 'C:\Program Files\Altiris\Altiris Agent\Agents\SoftwareManagement\Software Delivery\' -recurse | where {$_.name -like '*2016*'}
cmd /c 'C:\Program Files\Altiris\Altiris Agent\Agents\SoftwareManagement\Software Delivery\{F734ED03-43DA-4720-AB96-33739378FE9F}\cache\install.bat'

Cscript.exe "C:\Packages\OffScrub_O15msi.vbs" ALL /Quiet /NoCancel /Force /OSE

Get-WmiObject win32_product |  Where-Object {$_.Name  -like "Microsoft Office Professional*" -or $_.Name  -like "Microsoft Office Standard*"} | Select-Object  Name,Version

# Remote event logs:
Get-EventLog -LogName Application -InstanceId 1000 -Message *EXCEL.exe* | Select-Object -ExpandProperty message
Get-EventLog -LogName Application -InstanceId 1000 -Newest 5 | Select-Object -ExpandProperty message

# Remote patch installs:
cmd /c 'C:\Packages\csi2013-kb3172545-fullfile-x64-glb.exe' /passive /quiet
```

```
## To Be Orged:

# Basic .XML file needed for office removal:
# App ID: Access, PrjStd, Visio, Standard, Lync, SkypeforbusinessRetail, ACCESSRT

New-Item -Path 'C:\Packages\office.xml' | add-content -value "<Configuration Product=""Access"">`r`n<Display Level=""none"" CompletionNotice=""No"" SuppressModal=""Yes"" NoCancel=""Yes"" AcceptEula=""Yes"" />`r`n<Setting Id=""SETUP_REBOOT"" Value=""Never"" />`r`n</Configuration>"

Set-Content 'C:\Packages\office.xml' -value "<Configuration Product=""Access"">`r`n<Display Level=""none"" CompletionNotice=""No"" SuppressModal=""Yes"" NoCancel=""Yes"" AcceptEula=""Yes"" />`r`n<Setting Id=""SETUP_REBOOT"" Value=""Never"" />`r`n</Configuration>"

cmd /c '"C:\Program Files (x86)\Common Files\Microsoft Shared\OFFICE14\Office Setup Controller\setup.exe" /uninstall Standard /config c:\packages\office.xml'
cmd /c '"C:\Program Files\Common Files\Microsoft Shared\OFFICE14\Office Setup Controller\setup.exe" /uninstall Standard /config c:\packages\office.xml'

wmic product where "Vendor like '%Microsoft%'" get Name

cmd /c 'TASKKILL /F /IM outlook.exe & TASKKILL /F /IM lync.exe & TASKKILL /F /IM winword.exe & TASKKILL /F /IM excel.exe & TASKKILL /F /IM powerpnt.exe & TASKKILL /F /IM msaccess.exe & TASKKILL /F /IM onenote.exe & TASKKILL /F /IM groove.exe & TASKKILL /F /IM visio.exe & TASKKILL /F /IM winproj.exe & TASKKILL /F /IM MSOSYNC.EXE & TASKKILL /F /IM Teams.exe & TASKKILL /F /IM UcMapi.exe'

Cscript.exe "C:\Packages\OffScrub_O15msi.vbs” ALL /Quiet /NoCancel /Force /OSE
Cscript.exe "C:\Packages\OffScrub10.vbs” ALL /Quiet /NoCancel /Force /OSE
```

## Office Installation:
### "Error 1907. Could not register font. Verify that you have sufficient permissions to install fonts, and that the system supports this font."
This error is fixed by running "SFC /SCANNOW" which will resolve a file system issue, the command below is intended for a remote powershell session.
```
Start-Process -FilePath "${env:Windir}\System32\SFC.EXE" -ArgumentList '/scannow' -Wait -Verb RunAs -WindowStyle hidden
```

### "Error 1305. Setup cannot read file C:\Program Files (x86)\Common File\Microsoft Shared\OFFICE14\MSO.dll"

The error is related to the cd/dvd drive on the client. The upper filter go between the operating system and the main driver, while the lower driver go between the main driver and the hardware. Don't know exactly what causes the error but removing these keys, then rebooting the machine resolves the issue.

```
reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E967-E325-11CE-BFC1-08002BE10318} /v LowerFilters /f
reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E967-E325-11CE-BFC1-08002BE10318} /v UpperFilters /f
```

### "Error 30054. Unable to run setup.exe, when triggering install scrip error code is given"

*The language of this installation package is not supported by your system.*

The error is related to language pack compatability. The confusion in our case was that this was not related to langauage in the installation package, but the language in the OS. This user had a secondary language "Swedish (Finnish)" that was noted in the windows setting as missing languagae pack. Removing this secondary language allowed the install to continue and finish.

### "The file {90160000-0011-1000-0000000FF1CE}-C\ProPlusWW.msi could not be found. Setup failed. Rolling back changes..."

This error seems to be related to something with the language packs, initally it was thought that this was related to a corrupt install package but that was ruled out quite fast. The users system was set to english, but there must have been some issue with how that was registered on it when it was changed from one of the other languages. We worked around the issue by tweaking the confiuration file that is provided in our install packages, adjusted to pick english as default instead of trying to detect primary language.

```
# Config.XML
<AddLanguage Id="match" ShellTransform="yes"/>
<AddLanguage Id="en-us" />
# Edited to:
<AddLanguage Id="en-us" ShellTransform="yes"/>
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

### Tables, images, and attachments are missing in the received meeting requests in Outlook 2013 (sent from Office 2016)
*Assume that a meeting request that has table content, embedded images, and attachments is created in Microsoft Outlook 2016. Then, you receive the meeting request in Outlook 2013. In this situation, the tables, images, and attachments are missing or corrupted in the received meeting request.*

Resolved by installing the patch or editing the user registry:

- [Outlook 2013 KB3127975 Download](https://download.microsoft.com/download/6/1/5/615A5F76-7247-47BC-85BE-EB4CB30A05EF/outlook2013-kb3127975-fullfile-x64-glb.exe)

```
reg add "HKEY_CURRENT_USER\Software\Microsoft\Office\15.0\Outlook\Options\Calendar" /v AllowHTMLCalendarContent /t REG_DWORD /d 1 /f
```

### Email rule - run a script
This feature was by default hidden after a security update, it can be enabled through the users registry.

```
reg add "HKEY_CURRENT_USER\Software\Microsoft\Office\XX.X\Outlook\Security" /v EnableUnsafeClientMailRules /t REG_DWORD /d 1 /f
```

### The Delegates settings were not saved correctly. Unable to activate send-on-behalf-of list. You do not have sufficient permission to perform this operation on this object.

*This error can occur when the Delegates list contains a mailbox user who no longer exists in the organization. To fix the error remove the non-existent user from the Delegates list before you attempt to add other Delegates or change the Delegates settings.*

The first command is to check the attribute, the next following 2 are either to clear a specific inactive user or to remove them all.

```
Get-ADUser $user -Properties publicDelegates | Select-Object -ExpandProperty publicDelegates

Set-ADUser -Remove @{PublicDelegates="UID"}
Set-ADUser -clear PublicDelegates
```

### Items deleted in a shared mailbox goes to the users deleted items instead of the shared mailbox deleted items.

This option can be changed through the user registries. By default the key is set to 8, to store deleted items in the mailbox it was deleted from the key needs to be set to 4.

```
reg add "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office\XX.0\Outlook\Options\General" /v DelegateWastebasketStyle /t REG_DWORD /d 4 /f
```

### HTTPS linked images in HTML emails display the red X: "The linked image cannot be displayed.  The file may have been moved, renamed, or deleted.  Verify that the link points to the correct file.

This problem occurs when the Internet Explorer Security setting Do not save encrypted pages to disk option is enabled. This can be disabled per user or system wide.

```
reg add "HKEY_CURRENT_USER\software\microsoft\windows\CurrentVersion\Internet Settings" /v DisableCachingOfSSLPages /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\software\microsoft\windows\CurrentVersion\Internet Settings" /v DisableCachingOfSSLPages /t REG_DWORD /d 0 /f
```

### Issues when trying to open email attachments

Try to clear the users outlook temp folder and see if that resolves the issue

```
Remove-Item "$env:localappdata\Temporary Internet Files\Content.Outlook\" -confirm:$false -recurse
```

### Search indexing stuck and wont rebuild

By default windows search is responsible for search within outlook, we have had some cases where this have not been functioning properly and with the changes here we can change the search feature over to outlooks own built-in search. This works better for some users in our environment.

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v PreventIndexingOutlook /t REG_DWORD /d 1 /f
```

### Email list autocomplete not working

*autocomplete cache has a limit of 1000 entries, after that it starts dropping email addresses off as you add new ones and sometimes fails to add new ones*

The commands will clear the email cache locally and on the server side, then the affected user can test and see how the autofill runs.

```
Remove-Item "$env:userprofile\AppData\Local\Microsoft\Outlook\RoamCache\Stream_Autocomplete*" -confirm:$false
Outlook /cleanautocompletecache
```

*The cache will only repopulate with the addresses you actually send emails to, if you just load users in the "To" field and close the mail then the addresses will stick until you close outlook and then disappear from cache.*

## Excel:
### App v16.0.4978.1000 crashing when saving file to sharepoint

This issue appears with some files that are opened from sharepoint. This issue causes Excel 2016 to crash when you attempt to save the file, so the data gets lost instead of uploaded in to sharepoint.

*Issue appears to be related to the patches KB3191922/KB3203477*

The issue is resolved by installing the appropiate patch:

- [Microsoft Office 2016 - KB3213549 Download](https://download.microsoft.com/download/C/E/3/CE33320D-1400-494F-91E9-14666FD5D979/mso2016-kb3213549-fullfile-x64-glb.exe)

```
Faulting application name: EXCEL.EXE, version: 16.0.4978.1000, time stamp: 0x5e451d6b  
Faulting module name: EXCEL.EXE, version: 16.0.4978.1000, time stamp: 0x5e451d6b  
Exception code: 0xc0000005  
Fault offset: 0x00000000012d9fe9  
Faulting process id: 0x3afc  
Faulting application start time: 0x01d6ce279463d146  
Faulting application path: C:\Program Files\Microsoft Office\Office16\EXCEL.EXE  
Faulting module path: C:\Program Files\Microsoft Office\Office16\EXCEL.EXE  
Report Id: 00135a9e-d418-4adf-9e1c-f27105d52a3b
```

### App v16.0.5026.1000 crashing when saving file with macro
The issue is resolved by installing the appropiate patch or adding a user registry key:

- [Microsoft Excel 2016 - KB3085435 Download](https://download.microsoft.com/download/F/0/5/F05EABEE-70A2-423C-9F91-8AC3B5C65BB3/excel2016-kb3085435-fullfile-x64-glb.exe)
- reg add "HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Options" /v ForceVBALoadFromSource /t REG_DWORD /d 1 /f

```
Faulting application name: EXCEL.EXE, version: 16.0.5026.1000, time stamp: 0x5ed683b7
Faulting module name: VBE7.DLL, version: 7.1.10.97, time stamp: 0x5ea725f5
Exception code: 0xc0000005
Fault offset: 0x00000000001185ca
Faulting process id: 0x2050
Faulting application start time: 0x01d6c4cb8b20c758
Faulting application path: C:\Program Files\Microsoft Office\Office16\EXCEL.EXE
Faulting module path: C:\Program Files\Common Files\Microsoft Shared\VBA\VBA7.1\VBE7.DLL
Report Id: faec6dbe-fc1f-4346-9356-12d750d16502
```

### App v16.0.4978.1000 crashing with error reference to chart.dll
The issue is caused by KB4018319 and is solved by installing the appropriate patch:

- [Microsoft Office 2013 - KB2986229 Download](https://download.microsoft.com/download/B/9/9/B99D8FC5-569A-42F0-B0AD-089BB246C327/oart2013-kb2986229-fullfile-x64-glb.exe)
- [Microsoft Office 2016 - KB4011128 Download](https://download.microsoft.com/download/1/A/6/1A69A731-CEC8-4D3F-BA81-0E801E8B0C7D/chart2016-kb4011128-fullfile-x64-glb.exe)

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

### App v15.0.5233.1000 spreadsheets with macros crashing with error reference to gdi32full.dll

Not sure under what circumstances this issue is triggered but this case was on a Windows 10 v/1809 machine. Issue relates to some windows update released by microsoft, and is resolved by installing a patch based on the OS version.

- [Reference Article](https://kb.parallels.com/en/125027)

- [Windows 10 v1909 - KB4567512 Download](http://download.windowsupdate.com/c/msdownload/update/software/updt/2020/06/windows10.0-kb4567512-x64_2ea636c671529de2154d48a1181c0f02cd919da5.msu)
- [Windows 10 v1809 - KB4567513 Download](http://download.windowsupdate.com/c/msdownload/update/software/updt/2020/06/windows10.0-kb4567513-x64_64dba2ab8d4335fe82a5494bed86f20fd0de3b37.msu)

```
Faulting application name: EXCEL.EXE, version: 15.0.5233.1000, time stamp: 0x5e76d6d3
Faulting module name: gdi32full.dll, version: 10.0.17763.1282, time stamp: 0xf9833b72
Exception code: 0xc0000005
Fault offset: 0x0000000000079de0
Faulting process id: 0x72c
Faulting application start time: 0x01d64df834c8d737
Faulting application path: C:\Program Files\Microsoft Office\Office15\EXCEL.EXE
Faulting module path: C:\WINDOWS\System32\gdi32full.dll
Report Id: 56c03d41-c5b7-4561-952e-2d6a64c7d8df
```

### App v15.0.5249.1000 spreadsheets from shared drive crashing when trying to open or save

*When you open a document in Word, Excel, or PowerPoint 2013, a failure to open the document would occur repeatedly.*

- [Windows 10 v1909 - KB3172545 Download](https://download.microsoft.com/download/C/7/5/C752F185-509E-470E-934E-90ACB996B695/csi2013-kb3172545-fullfile-x64-glb.exe)

```
Faulting application name: EXCEL.EXE, version: 15.0.5249.1000, time stamp: 0x5ebb2bed
Faulting module name: EXCEL.EXE, version: 15.0.5249.1000, time stamp: 0x5ebb2bed
Exception code: 0xc0000005
Fault offset: 0x00000000001cdfc2
Faulting process id: 0x2f5c
Faulting application start time: 0x01d6f97841d69e29
Faulting application path: C:\Program Files\Microsoft Office\Office15\EXCEL.EXE
Faulting module path: C:\Program Files\Microsoft Office\Office15\EXCEL.EXE
Report Id: bdf34643-593a-4acd-aec5-e7da65d9d940
```

### This file cannot be previewed because there is no previewer installed for it.

This issue is supposedly caused by some Office 2016 patch, there were some machines that had some software changes which caused the fix to never get installed so on these machines the reg key value {00020827-0000-0000-C000-000000000046} did exist, but the registry type was set to REG_EXPAND_SZ and was pointing to a .dll file in the Office16 common files folder. Resolved by creating the key with the proper values

```
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\PreviewHandlers" /v "{00020827-0000-0000-C000-000000000046}" /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\PreviewHandlers" /v "{00020827-0000-0000-C000-000000000046}" /t REG_SZ /d "Microsoft Excel previewer" /f
```

### Smart View: http session timeouts

The session settings are set per user, this isssue is generally resolved by increasing the timeout time. The changes below is to set the timeout timer to 900000 ms which is 15 minutes

```
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ReceiveTimeout /t REG_DWORD /d 900000 /f
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v KeepAliveTimeout /t REG_DWORD /d 900000 /f
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ServerInfoTimeout /t REG_DWORD /d 900000 /f
```

### Wont open files from shared drive, displays blank page

Have not found an official fix for this issue, but we can work around it by closing all office apps and then renaming the user settings folder. When a office app is then opened then a new settings folder is created with the software defaults.

```
cmd /c 'TASKKILL /F /IM outlook.exe & TASKKILL /F /IM lync.exe & TASKKILL /F /IM winword.exe & TASKKILL /F /IM excel.exe & TASKKILL /F /IM powerpnt.exe & TASKKILL /F /IM msaccess.exe & TASKKILL /F /IM onenote.exe & TASKKILL /F /IM groove.exe & TASKKILL /F /IM visio.exe & TASKKILL /F /IM winproj.exe & TASKKILL /F /IM MSOSYNC.EXE & TASKKILL /F /IM Teams.exe'
Rename-Item "$env:LOCALAPPDATA\Microsoft\Office" "$env:LOCALAPPDATA\Microsoft\Office.old"
```

### Application freeze when copying data from spreadsheet to spreadsheet

Difficult to tell exactly why this happens, it can be when copying a lot or a tiny amount of date from experience. There are a bunch of settings that can be tweaked which i have found to help a lot with the spreadsheet performance.

```
Options > General > User Interface options > enable live preview | untick the option
Options > Advanced > Cut, copy, and paste > Show Paste Options button when content is pasted | untick the option
Options > Advanced > Cut, copy, and paste > Show Insert Options buttons | untick the option
Options > Advanced > Cut, copy, and paste > Cut, copy, and sort inserted objects with their parent cells | untick the option
Options > Advanced > display > disable hardware graphics acceleration | tick the option

Options > Trust Center > Trust Center Settings > Protected View > untick everything
Options > Trust Center > Trust Center Settings > Macro Settings | make sure disable all macros with notification is ticked
Options > Trust Center > Trust Center Settings > Macro Settings > Trust access to the VBA project object model | tick the option
Options > Trust Center > Trust Center Settings > ActiveX Settings | make sure prompt me before enabling all controls with minimal restrictions is ticked
```

### Application error when trying to open an embedded .XLSX from a Word 2016 document

*The server application, source file, or item cannot be found. Make sure the application is properly installed, and that it has not been deleted, moved, renamed, or blocked by policy.*

```
# Issue was .xlsx was set to "xlsx_auto_file" instead of "Excel.Sheet.12"
Computer\HKEY_CLASSES_ROOT\.xlsx - (Default) - Excel.Sheet.12
# Alternative
HKEY_CURRENT_USER\SOFTWARE\Classes\.xlsx - (Default) - Excel.Sheet.12
```

## PowerPoint:
### App v16.0.4266.1001 powerpoints crashing with error reference to ppcore.dll

*When you select the Font tab of the graph edit dialog box in an Office 2016 application, such as Word 2016, Excel 2016 or PowerPoint 2016, the Office 2016 application crashes.*

The issue is resolved by installing one or both of these patches, the first one is the most likely candidate, the second update contains some security fixes that does apply an update to ppcore.dll as well. Not had the chance to test each of them individually yet.

- [Windows 10 v1909 - KB4011211 Download](https://download.microsoft.com/download/2/0/9/2091826A-3C42-4771-980F-E865265EA00C/graph2016-kb4011211-fullfile-x64-glb.exe)
- [Windows 10 v1909 - KB4011041 Download](https://download.microsoft.com/download/E/5/1/E5117530-5B8F-4D46-A65F-0E4BD3F4A0B5/powerpoint2016-kb4011041-fullfile-x64-glb.exe)

```
Faulting application name: POWERPNT.EXE, version: 16.0.4266.1001, time stamp: 0x55ba17c4
Faulting module name: ppcore.dll, version: 16.0.4993.1001, time stamp: 0x5e452539
Exception code: 0xc0000005
Fault offset: 0x0000000000a58ffc
Faulting process id: 0x3d7c
Faulting application start time: 0x01d6f4ca9963e304
Faulting application path: C:\Program Files\Microsoft Office\Office16\POWERPNT.EXE
Faulting module path: C:\Program Files\Microsoft Office\Office16\ppcore.dll
Report Id: 1b25592f-68ab-4e0a-96f0-f65b2a1d538a
```

## Word:
### App v16.0.5056.1000 unable to open or save Word documents

The issue seems to be related to KB4011730 and is resolved by installing the patch KB4018295.

- [KB4018295 Download](https://download.microsoft.com/download/1/8/8/188E5F6B-4B94-41FF-AD18-6A825C22F162/mso2016-kb4018295-fullfile-x64-glb.exe)

```
Faulting application name: WINWORD.EXE, version: 16.0.5056.1000, time stamp: 0x5f32dc1c
Faulting module name: wwlib.dll, version: 16.0.5056.1000, time stamp: 0x5f32db69
Exception code: 0xc000041d
Fault offset: 0x0000000000c5796d
Faulting process id: 0x3a94
Faulting application start time: 0x01d6f88c19bb9971
Faulting application path: C:\Program Files\Microsoft Office\Office16\WINWORD.EXE
Faulting module path: C:\Program Files\Microsoft Office\Office16\wwlib.dll
Report Id: 041752fd-0660-4989-b646-d38d3e19eeff
```

## Visio
### App v15.0 "Please wait while Windows configures Microsoft Visio Professional 2013"

In this case the app was running perfectly fine as another user, if that is the case then the issue will be in the specific users classes root registries. Check and change the setting accordingly and the configuration prompt will stop.

```
Issue was .vsdx was set to "vsdx_auto_file" instead of "Visio.Drawing.15"

Computer\HKEY_CLASSES_ROOT\.vsd - (Default) - Visio.Drawing.11
Computer\HKEY_CLASSES_ROOT\.vsdx - (Default) - Visio.Drawing.15
```

## OneNote:
### Microsoft OneNote 2013 requires Visual Basic for Applications. If you continue, Visual Basic for Applications will be installed.

This issue is due to some old files being left behind from a previous installation and the installer is unable to override these, so this is why the application tries to configure these on each launch. In this case the client had gone through an upgrade from 2010 to 2013.

```
rename-Item "C:\Program Files\Common Files\Microsoft Shared\VBA" "C:\Program Files\Common Files\Microsoft Shared\VBA.old" -confirm:$false -force

Set-Content 'C:\Packages\office.xml' -value "<Configuration Product=""Standard"">`r`n<Display Level=""none"" CompletionNotice=""No"" SuppressModal=""Yes"" NoCancel=""Yes"" AcceptEula=""Yes"" />`r`n<Setting Id=""SETUP_REBOOT"" Value=""Never"" />`r`n</Configuration>"

cmd /c '"C:\Program Files\Common Files\Microsoft Shared\OFFICE15\Office Setup Controller\setup.exe" /repair Standard /config c:\packages\office.xml'
```

### "We couldn't find a notebook at "https://url". A notebook usually has a table of contents file (*.onetoc or *.onetoc2)."

This error was given for a onenote book that is hosted in a sharepoint site. Researching online they hint that the issue could be related to a sharepoint feature, but that seemed odd since it was working some days before. The user tested and ruled that out, what we found though was that this pc somehow had the windows 10 store onenote app, the file was then attempting to open with that app instead of the 2016 desktop application. Issue was resolved by removing the store app.

```
# Open CMD as the user
Get-AppxPackage *OneNote* | Remove-AppxPackage
# Alternative to try
Under Site Collection Administrator > Site Collection features, locate "Limited-access user permission lockdown mode, and then select the "Deactivate" button."
```

## SharePoint Designer 2013
### Server-side activities have been updated. You need to restart SharePoint Designer to use the updated version of activities.

*When you try to create a SharePoint 2013 workflow in SharePoint Designer 2013 on a computer that has Microsoft.Activities.dll installed, you receive the following error message.*

The application doesn't need to have everything installed in a specific order, but SP1 and KB3114337 are mandatory. After KB3114337 is installed, the cache needs to be completely cleared.

- [SharePoint Designer 2013 Download](https://download.microsoft.com/download/3/E/3/3E383BC4-C6EC-4DEA-A86A-C0E99F0F3BD9/sharepointdesigner_64bit.exe)
- [SP1 KB2817441 Download](https://download.microsoft.com/download/F/C/7/FC7EC278-E333-450B-89AF-4F9640E73D62/spdsp2013-kb2817441-fullfile-x64-en-us.exe)
- [KB2863836 Download](https://download.microsoft.com/download/9/D/A/9DA5C7EB-8BEA-45B4-A055-D5977B497017/spd2013-kb2863836-fullfile-x64-glb.exe)
- [KB3114337 Download](http://download.windowsupdate.com/d/msdownload/update/software/crup/2016/01/spd-x-none_7c1009a5a70cac8d7012a44d2710faf79d7d7fb5.cab)
- [KB3114721 Download](https://download.microsoft.com/download/6/2/3/623ABAA1-836E-4BBC-BDC5-F7DF8BA589F8/spd2013-kb3114721-fullfile-x64-glb.exe)

```
# C:\Users\*\appdata\roaming\microsoft\SharePoint Designer\ProxyAssemblyCache
Get-ChildItem -Path "C:\Users\*\appdata\roaming\microsoft\SharePoint Designer\ProxyAssemblyCache" -recurse -Force | remove-item -confirm:$false -recurse -Force -ErrorAction SilentlyContinue
# C:\Users\*\appdata\Roaming\Microsoft\Web Server Extensions\Cache
Get-ChildItem -Path "C:\Users\*\appdata\Roaming\Microsoft\Web Server Extensions\Cache" -recurse -Force | remove-item -confirm:$false -recurse -Force -ErrorAction SilentlyContinue
# C:\Users\*\appdata\local\microsoft\websitecache
Get-ChildItem -Path "C:\Users\*\appdata\local\microsoft\websitecache" -recurse -Force | remove-item -confirm:$false -recurse -Force -ErrorAction SilentlyContinue
```

## Microsoft Teams
### Client Downloads

- [MS Teams: v/1.3.00.33674 Download](https://statics.teams.cdn.office.net/production-windows-x64/1.3.00.33674/Teams_windows_x64.exe)
- [MS Teams: v/1.3.00.32283 Download](https://statics.teams.cdn.office.net/production-windows-x64/1.3.00.32283/Teams_windows_x64.exe)
- [MS Teams: v/1.3.00.30866 Download](https://statics.teams.cdn.office.net/production-windows-x64/1.3.00.30866/Teams_windows_x64.exe)
- [MS Teams: v/1.3.00.28779 Download](https://statics.teams.cdn.office.net/production-windows-x64//Teams_windows_x64.exe)
- [MS Teams: v/1.3.00.26064 Download](https://statics.teams.cdn.office.net/production-windows-x64/1.3.00.26064/Teams_windows_x64.exe)

## Skype for Business
### Webcam freezing after windows update

*After the anniversary update Microsoft dropped support for MJPEG and H264 encoding standards to move towards YUY2 encoding for performance. The change lead to an issue that Webcams that use MJPEG or H264 would not work correctly. This issue then came up again after an later update as well*

To resolve add the registries, then re-open the affected app or restart pc.

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Media Foundation\Platform" /v EnableFrameServerMode /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows Media Foundation\Platform" /v EnableFrameServerMode /t REG_DWORD /d 0 /f
```

### No emoji icons visible, the IM is just blank with rectangle box

*When users send emojis, they do not see the emoji or receive it from others.

```
reg add "HKEY_CURRENT_USER\Software\Microsoft\Office\XX.0\Lync" /v DisableRicherEditCanSetReadOnly /t REG_DWORD /d 1 /f
```

### Precense icons in outlook not working

```
reg add "HKEY_CURRENT_USER\SOFTWARE\IM Providers" /v DefaultIMApp /d Lync /f
```

### Skype meeting button missing in outlook calendar

Resolved by clearing the users office registry key, any manual changes done in the registry to try and force the plugin to load for some reason does not work. This will reset all the user settings for Office.

```
reg delete "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office" /f
```

### App crashing when you click on the toaster notification for any new IM or incoming calls

*Issue appeared after KB4011159 - resolved by installing one of the patches here*

- [Microsoft Office 2016 KB4011631 Download](https://download.microsoft.com/download/E/3/6/E36DCF64-73A4-430E-9E5B-107CE49F714F/msodll40ui2016-kb4011631-fullfile-x64-glb.exe)
- [Microsoft Office 2016 KB4011099 Download](https://download.microsoft.com/download/B/A/1/BA1D3A64-F1AD-49AB-B80E-2D8367D8ACBB/msodll40ui2016-kb4011099-fullfile-x64-glb.exe)

## Access Runtime
### App v2016: "Error 2711.  An internal error has occurred.  (ACCESSFiles)."

*Automatic configuration of the current version of Microsoft Access has failed. Your database might not function correctly. This can occur if you do not have the necessary permissions to install Microsoft Access on this computer.*

The issue here can occur on machines that had previous versions of access runtime installed. Under the following registry location there is a default variable, that is pointing to ACEOLEDB.DLL and if this path is incorrect then you will get this error.

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{3BE786A0-0366-4F5C-9434-25CF162E475E}\InprocServer32" /ve /t REG_SZ /d "C:\Program Files\Common Files\Microsoft Shared\OFFICE16\ACEOLEDB.DLL" /f
```

## Office
### Disable Office Sync Tool

The only way to stop this tool from running on boot is to remove the user registries under run.

```
# Disable the sync tool
reg delete "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /V OfficeSyncProcess /F

# Re-enable the sync tool - 2013
reg add "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v OfficeSyncProcess /t REG_SZ /d "\"C:\Program Files\Microsoft Office\Office15\MSOSYNC.EXE"\" /f
```

### Onedrive not running automatically

```
# CMD as the user
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /f /v "OneDrive" /t REG_SZ /d "\"%LOCALAPPDATA%\Microsoft\OneDrive\OneDrive.exe\" /background"
```





