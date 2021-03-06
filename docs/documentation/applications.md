---
layout: default
title: Applications
parent: Documentation
nav_order: 3
---

{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Browsers
### Internet Explorer: This website wants to install the following add-on "crystal report activex viewer control"

> The: ActiveXViewer.cab file needs to be downloaded from the source
> http://yourwebsitewherecrystalisinstalled/crystalreportviewers11/ActiveXControls/ActiveXViewer.cab

Extract ActiveXViewer.cab to a folder on the machine, then manually register the .DLL's, restart browser and access the page again.

```
REGSVR32 /S CRVIEWER.DLL
REGSVR32 /S REPORTPARAMETERDIALOG.DLL
REGSVR32 /S SVIEWHLP.DLL
REGSVR32 /S SWEBRS.DLL
```


### Internet Explorer: DLG_FLAGS_INVALID_CA "Your PC doesn’t trust this website’s security certificate."

*Because this site uses HTTP Strict Transport Security, you can’t continue to this site at this time.*

This error is due to issues regarding Zscaler's root CA certificate, this needs to be downloaded and installed again.

- [Zscaler root CA Certificate](http://keyserver.xxx.com/pki/X3/ZscalerRootCertificate-2048-SHA256.crt)

Download the certificate. Double click it and install for user, then do the same and install for machine, and lastly restart Internet Explorer.

### Google Chrome: "The application has failed to start because its side-by-side configuration is incorrect."

This error is generally resolved by quickly re-installing google chrome, in our environment there are cases where removing/upgrading chrome fails because the old installation reference is still in the registry. So instead of installing as it should the package delivery system asks for the old installer to remove the old app. In the registry location below there should be a google chrome reference, if it is deleted then the software can be pushed.

```
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\KeyID
```

### Google Chrome/Firefox: To export a list you must have a Microsoft SharePoint Foundation compatible application

This is an error message you will receive if you are trying to export sharepoint lists in firefox/chrome, this is due that both these browsers doesn't support ActiveX controls which is used in sharepoint 2013 to validate if you have excel installed, this controller is called SpreadSheetLauncher. For this feature use Internet Explorer.

### Google Chrome: corrupt user profile

```
Rename-Item "$env:LOCALAPPDATA\Google\Chrome\User Data\Default" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default.old" -confirm:$false
Copy-Item "$env:LOCALAPPDATA\Google\Chrome\User Data\Default.old\bookmarks" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\" -confirm:$false
```

### Google Chrome: displays window in all white or black

Open run and paste in the command below, if it runs sucessfully update the graphics driver on the computer

```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disable-gpu
```

### Update for Removal of Adobe Flash Player for Windows

*Applying this update will remove Adobe Flash Player from your Windows device. After this update has been applied, this update cannot be uninstalled.*

This patch is designed to remove the adobe flash player that is built in to Microsoft Edge and Internet Explorer, the patch will only remove the built in version and wont affect any other standalone installations.

- [Windows 10 v1909 KB4577586 Download](http://download.windowsupdate.com/c/msdownload/update/software/updt/2020/10/windows10.0-kb4577586-x64_ec16e118cd8b99df185402c7a0c65a31e031a6f0.msu)

## Java
### The following resource is signed with a weak signature algorithm MD5withRSA and is treated as unsigned.

As of java 8u131 applications signed with MD5withRSA/DSA algorithms are treated as unsigned. To bypass this modify the java.security file in the program files folder to still allow the algorithm.

```
((Get-Content -path "C:\Program Files (x86)\Java\jre1.8.*\lib\security\java.security") -replace 'jdk.jar.disabledAlgorithms=MD2, MD5, RSA keySize < 1024','#jdk.jar.disabledAlgorithms=MD2, MD5, RSA keySize < 1024') | Set-Content -Path "C:\Program Files (x86)\Java\jre1.8.*\lib\security\java.security"
```

### Can not verify Deployment Rule Set jar due to certificate expiration

```powershell
remove-item 'C:\Windows\Sun\Java\Deployment\DeploymentRuleSet.jar' -confirm:$false -force
```

### Disable java update prompt



```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\JavaSoft\Java Update\Policy" /v NotifyDownload /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\JavaSoft\Java Update\Policy" /v EnableJavaUpdate /t REG_DWORD /d 0 /f
```

### Viewing oracle app and getting: java.lang.ClassNotFoundException: oracle.forms.engine.Main

The issue in our case affected the certificate store, which then errored out the java applet. Quickly solved by removing the cacerts file.

```
Remove-Item "C:\Program Files (x86)\Java\jre*\lib\security\cacerts" -confirm:$false -force
```

## Cisco AnyConnect
### Unable to change domain password in windows 10

```
Logon Denied
Only one user session is allowed.
%user% is already logged onto this machine.

This user must log off of this machine before you can log on.
```

Issue is resolved by changing a registry key, unable to determine why this was and issue for just a few clients.

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{B12744B8-5BB7-463a-B85E-BB7627E73002}" /v EnforceSingleLogon /t REG_DWORD /d 0 /f
```

## Citrix Reciever/WorkSpaceApp
### The remote session was disconnected because there are no Terminal Service License Servers available to provide a license. Please contact your server administrator

```
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\MSLicensing" /f
```

### WorkSpaceApp v1904: Unable to connect to the server. Contact your system administrator with the following error: SSL Error 47: The server sent an SSL alert: sslv3 alert handshake failure (alert number unavailable)

*Note: Citrix has deprecated weak cryptography across the board. If the configurations on the backend is not updated to support one of the 3 supported strong cipher suites, you will not be able to connect.*

- [Overview of the Crypto Kit updates in Citrix Workspace for Windows and Mac](https://support.citrix.com/article/CTX250104)

```
# At least one of these is required:  
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)  
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)  
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)
```

## Applications
### Notepad++ "When starting app it is not responding"

This issue was resolved by removing the session.xml from the users profile, then it should work once you open the app again.

```
Remove-Item "C:\Users\$env:username\AppData\Roaming\Notepad++\session.xml" -confirm:$false
```

### Anixis PPE - Password Policy Enforcer - Unable to change password while on VPN

*The server SERVERXX.domain.com did not respond (10060). Make sure this server is running PPE V9.0 or later and UDP Port 1333 is not blocked by a firewall.*

> Vendor confirmed that the Anixis tool requires a open port to a DC, either there is none set up or there is some issue with connectivity towards the DC.




