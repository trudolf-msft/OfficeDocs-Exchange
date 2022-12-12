---
ms.localizationpriority: medium
ms.topic: article
author: lusassl-msft
ms.author: lusassl
title: Exchange Server TLS configuration best practices
description:  "Learn about Exchange Server TLS configuration best practices."
ms.prod: exchange-server-it-pro
manager: serdars
---

# Exchange Server TLS configuration best practices

This documentation describes the required steps to properly configure TLS 1.2 on Exchange Server 2013, Exchange Server 2016 and Exchange Server 2019. It also describes how to optimize the cipher suites and hashing algorithms used by TLS 1.2 (**Exchange Server 2016 only**). Furthermore, it describes how to properly configure TLS 1.0 and 1.1 whether you want it disabled and configured correctly within .NET Framework. Please read carefully as some of the steps described here can only be performed on specific operating systems (like Windows Server 2016) or specific Exchange Server versions.

> [!NOTE]
> The [Microsoft TLS 1.0 implementation](https://support.microsoft.com/topic/schannel-implementation-of-tls-1-0-in-windows-security-status-update-november-24-2015-69b482ff-072d-f8a8-1ba3-e921019a4d5f) has no known security vulnerabilities. But because of the potential for future protocol downgrade attacks and other TLS vulnerabilities, it is recommended to carefully plan and disable TLS 1.0 and 1.1. Failure to plan carefully may cause clients to lose connectivity.

> [!IMPORTANT]
> This document contains steps that tell you how to modify the registry. However, serious problems might occur if you modify the registry incorrectly. Therefore, make sure that you follow these steps carefully. For added protection, back up the registry before you modify it. Then, you can restore the registry if a problem occurs. For more information about how to back up and restore the registry, see [How to back up and restore the registry in Windows](https://support.microsoft.com/topic/how-to-back-up-and-restore-the-registry-in-windows-855140ad-e318-2a13-2829-d428a2ab0692).

## Prerequisites

TLS 1.2 support was added with Cumulative Update (CU) 19 to Exchange Server 2013 and CU 8 to Exchange Server 2016. Exchange Server 2019 supports TLS 1.2 out of the box. It is possible to disable TLS 1.0 and 1.1 on Exchange Server 2013 with CU 20 and later or on Exchange Server 2016 with CU 9 and later. It is also required to have the latest version of .NET Framework and associated patches [supported by your CU](/exchange/plan-and-deploy/supportability-matrix?view=exchserver-2016#exchange-2016&preserve-view=true) in place.

Exchange Server cannot run without Windows Server therefore it is important to have the latest operating system updates installed to run a stable and secure TLS 1.2 implementation.

Based on your operating system, please make sure that the following updates are also in place (they should be installed if your server is current on Windows Updates):

<br>

****

|Operating System|Required Updates|
|---|---|
|Windows Server 2016|N/A|
|Windows Server 2012 (R2)|[KB3161949](https://support.microsoft.com/topic/ms16-077-description-of-the-security-update-for-wpad-june-14-2016-0d3aee51-dbee-bfc9-fbf3-201178b51914), [KB2973337](https://support.microsoft.com/topic/sha512-is-disabled-in-windows-when-you-use-tls-1-2-5863e74e-e5b6-cc3b-759b-ece8da875825)|

## Enabling TLS 1.2

### Enable TLS 1.2 for Schannel

> [!NOTE]
> When configuring a system for TLS 1.2, you can make the Schannel and .NET registry keys at the same time and reboot the server once.

1. From Notepad.exe, create a text file named **TLS12-Enable.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    ```
3. Save **TLS12-Enable.reg**
4. Double-click the **TLS12-Enable.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Enable TLS 1.2 for .NET 4.x

The ``SystemDefaultTlsVersions`` registry value defines which security protocol version defaults will be used by .NET Framework 4.x. If the value is set to 1, then .NET Framework 4.x will inherit its defaults from the Windows Schannel ``DisabledByDefault`` registry values. If the value is undefined, it will behave as if the value is set to 0. The strong cryptography (configured by the ``SchUseStrongCrypto`` registry value) uses more secure network protocols (TLS 1.2, TLS 1.1, and TLS 1.0) and blocks protocols that are not secure. ``SchUseStrongCrypto`` affects only client (outgoing) connections in your application. By configuring .NET Framework 4.x to inherit its values from Schannel we gain the ability to use the latest versions of TLS supported by the OS, including TLS 1.2.

> [!NOTE]
> [Exchange Server 2019 comes with a TLS 1.2 only default configuration](/exchange/new-features/new-features?view=exchserver-2019#security&preserve-view=true). Most of the steps described in this article are already configured. However, `SchUseStrongCrypto` is currently not configured by default and should therefore be configured manually as described below. Microsoft is investigating the possibility of adding this configuration in a future Exchange Server 2019 update.

1. From Notepad.exe, create a text file named **NET4X-UseSchannelDefaults.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto"=dword:00000001
    ```
3. Save **NET4X-UseSchannelDefaults.reg**
4. Double-click the **NET4X-UseSchannelDefaults.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Enable TLS 1.2 for .NET 3.5

Exchange Server 2013 and later do not need this anymore. However, we recommend to configure it identically to the .NET 4.x setting to ensure a consistent configuration.

1. From Notepad.exe, create a text file named **NET35-UseSchannelDefaults.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v2.0.50727]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v2.0.50727]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto"=dword:00000001
    ```
3. Save **NET35-UseSchannelDefaults.reg**
4. Double-click the **NET35-UseSchannelDefaults.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

## Validating TLS 1.2 is in use

Once TLS 1.2 has been enabled it may be helpful to validate your work was successful and the system is able to negotiate TLS 1.2 for inbound (server) connections and outbound (client) connections. There are a few methods available for validating this, some of them are discussed in the sections below.

Many protocols used in Exchange Server are HTTP based, and therefore traverse the IIS processes on the Exchange server. MAPI/HTTP, Outlook Anywhere, Exchange Web Services, Exchange ActiveSync, REST, OWA & EAC, Offline Address Book downloads, and AutoDiscover are examples of HTTP based protocols used by Exchange Server.

### Windows Server 2016 and Windows Server 2012 R2

The IIS team has added capabilities to Windows Server 2016 and Windows Server 2012 R2 to log custom fields related to encryption protocol versions and ciphers. We recommend reviewing the blog for documentation on [how to enable these custom fields](https://www.microsoft.com/security/blog/2017/09/07/new-iis-functionality-to-help-identify-weak-tls-usage/) and begin parsing logs for information on incoming connections in your environment related to HTTP based protocols.

These IIS custom fields do not exist for Windows Server 2012. Your load balancer or firewall logs may be able to provide this information. Please request guidance from your vendors to determine if their logs may provide this information.

### Message Headers (Exchange Server 2016 or later)

Message header data in Exchange Server 2016 provides the protocol negotiated and used when the sending and receiving host exchanged a piece of mail. You can use the [Message Header Analyzer](https://aka.ms/mha) to get a clear overview of each hop.

> [!NOTE]
> There is one known exception to the message headers example. When a client sends a message by connecting to a server using authenticated SMTP (also known as the SMTP client submission protocol), the TLS version in the messages headers does not show the correct TLS version used by a customer’s client or device. Microsoft is investigating the possibility of adding this information in a future update.

### Mail Flow via SMTP Logging

SMTP logs in Exchange Server 2013 and newer will contain the encryption protocol and other encryption related information used during the exchange of email between two systems.

When the server is the **SMTP receiving system**, the following strings exist in the log depending on the version of TLS used:

- TLS protocol SP_PROT_TLS1_0_SERVER
- TLS protocol SP_PROT_TLS1_1_SERVER
- TLS protocol SP_PROT_TLS1_2_SERVER

When the server is the **SMTP sending system**, the following strings exist in the log depending on the version of TLS used:

- TLS protocol SP_PROT-TLS1_0_CLIENT
- TLS protocol SP_PROT-TLS1_1_CLIENT
- TLS protocol SP_PROT-TLS1_2_CLIENT

Example to search in protocol log files:
```powershell
Select-String -Path (((Get-TransportService).ReceiveProtocolLogPath | select -First 1).PathName.Replace("Hub","FrontEnd")+"\*.log") "SP_PROT_TLS1_0"
```

### POP & IMAP

No logging exists which will expose the encryption protocol version used for POP & IMAP clients. To capture this information, you may need to capture Netmon logs from your server or inspect traffic as it flows through your load balancer or firewall where HTTPS bridging is taking place.

## Configure TLS 1.0 and 1.1 or disable them

> [!IMPORTANT]
> If you want to keep TLS 1.0 and TLS 1.1 enabled, please proceed with the steps described in the **Configure TLS 1.0 and 1.1** section and skip the **Disable TLS 1.0 and 1.1** section. If you want to disable TLS 1.0 and TLS 1.1, skip the **Configure TLS 1.0 and 1.1** section and continue with the **Disable TLS 1.0 and 1.1** section.

## Configure TLS 1.0 and 1.1

### Enable TLS 1.0 and 1.1 in Schannel

> [!NOTE]
> When configuring a system for TLS 1.0 and TLS 1.1 use, you can make the Schannel keys at the same time and reboot the server once.

An admin must modify the TLS 1.0 and TLS 1.1 portions of the Schannel registry section and turn the protocols on instead of turning them off.

To enable **TLS 1.0** for both Server (inbound) and Client (outbound) connections on an Exchange Server perform the following:

1. From Notepad.exe, create a text file named **TLS10-Enable.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    ```
3. Save **TLS10-Enable.reg**
4. Double-click the **TLS10-Enable.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

To enable **TLS 1.1** for both Server (inbound) and Client (outbound) connections on an Exchange Server perform the following:

1. From Notepad.exe, create a text file named **TLS11-Enable.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Client]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server]
    "DisabledByDefault"=dword:00000000
    "Enabled"=dword:00000001
    ```
3. Save **TLS11-Enable.reg**
4. Double-click the **TLS11-Enable.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

## Disable TLS 1.0 and 1.1

Please make sure that every application supports TLS 1.2 before disabling TLS 1.0 and 1.1. Considerations such as (but not limited to):

- Do your Domain Controllers and Global Catalog servers support TLS 1.2?
- Do partner applications (such as, but not limited to, SharePoint, Lync, Skype for Business, etc.) support TLS 1.2?
- Have you updated older Windows 7 desktops using Outlook to support [TLS 1.2 over WinHTTP](https://support.microsoft.com/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-a-default-secure-protocols-in)?
- Do your load balancers support TLS 1.2 being used?
- Do your desktop, mobile, and browser applications support TLS 1.2?
- Do devices such as multi-function printers support TLS 1.2?
- Do your third-party or custom in-house applications that integrate with Exchange Server or Office 356 support TLS 1.2?

As such we strongly recommend any steps you take to transition to TLS 1.2 and away from older security protocols are first performed in labs which simulate your production environments before you slowly start rolling them out in production.

The steps used to disable TLS 1.0 and 1.1 outlined below will apply to the following Exchange functionality:

- Simple Mail Transport Protocol (SMTP)
- Outlook Client Connectivity (Outlook Anywhere / Mapi over HTTP)
- Exchange Active Sync (EAS)
- Outlook on the Web (OWA)
- Exchange Admin Center (EAC) and Exchange Control Panel (ECP)
- AutoDiscover
- Exchange Web Services (EWS)
- REST (Exchange Server 2016 only)
- Use of PowerShell by Exchange over HTTPS
- POP and IMAP

### Disable TLS 1.0 and 1.1 in Schannel

> [!NOTE]
> When configuring a system for TLS 1.2 only use, you can make the Schannel keys at the same time and reboot the server once.

An admin must modify the TLS 1.0 and TLS 1.1 portions of the Schannel registry section and turn the protocols off instead of turning them on.

To disable **TLS 1.0** for both Server (inbound) and Client (outbound) connections on an Exchange Server perform the following:

1. From Notepad.exe, create a text file named **TLS10-Disable.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client]
    "DisabledByDefault"=dword:00000001
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server]
    "DisabledByDefault"=dword:00000001
    "Enabled"=dword:00000000
    ```
3. Save **TLS10-Disable.reg**
4. Double-click the **TLS10-Disable.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

To disable **TLS 1.1** for both Server (inbound) and Client (outbound) connections on an Exchange Server perform the following:

1. From Notepad.exe, create a text file named **TLS11-Disable.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Client]
    "DisabledByDefault"=dword:00000001
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server]
    "DisabledByDefault"=dword:00000001
    "Enabled"=dword:00000000
    ```
3. Save **TLS11-Disable.reg**
4. Double-click the **TLS11-Disable.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

## Cipher and hashing algorithms (Exchange Server 2016 only)

> [!IMPORTANT]
> The steps described in this section are optional to the steps described before. It's required to configure TLS 1.2 and fully disable TLS 1.0 and 1.1 before following the next steps.
>
> Consider applying these settings separate to disabling TLS 1.0 & TLS 1.1 to isolate configuration issues with problematic clients.

### Configure client and server TLS renegotiation strict mode

These settings are used to configure TLS renegotiation strict mode. This means that the server allows only those clients to which this [security update](https://support.microsoft.com/topic/ms10-049-vulnerabilities-in-schannel-could-allow-remote-code-execution-d4258037-ad3a-c00c-250f-6c67a408bd7c) is applied to set up and renegotiate TLS sessions. The server does not allow the clients to which this [security update](https://support.microsoft.com/topic/ms10-049-vulnerabilities-in-schannel-could-allow-remote-code-execution-d4258037-ad3a-c00c-250f-6c67a408bd7c) is not applied to set up the TLS session. In this case, the server terminates such requests from the clients.

Similarly, if this [security update](https://support.microsoft.com/topic/ms10-049-vulnerabilities-in-schannel-could-allow-remote-code-execution-d4258037-ad3a-c00c-250f-6c67a408bd7c) is applied to the client, and the client is in strict mode, the client can set up and renegotiate TLS sessions with all the servers for which this security update is applied. The clients cannot set up TLS sessions at all with servers for which this security update is not applied. The client cannot move ahead with a TLS negotiation attempt with such servers.

1. From Notepad.exe, create a text file named **ConfigureRenegotiation.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL]
    "AllowInsecureRenegoClients"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL]
    "AllowInsecureRenegoServers"=dword:00000000
    ```
3. Save **ConfigureRenegotiation.reg**
4. Double-click the **ConfigureRenegotiation.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Configure ciphers

We recommend explicitly disabling the following ciphers which are outdated and should not be used anymore.

1. From Notepad.exe, create a text file named **DisableCiphers.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\DES 56/56]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\NULL]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC2 40/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC2 56/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC2 56/56]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 40/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 56/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 64/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 128/128]
    "Enabled"=dword:00000000
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\Triple DES 168]
    "Enabled"=dword:00000000
    ```
3. Save **DisableCiphers.reg**
4. Double-click the **DisableCiphers.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Configure hashes

We recommend explicitly disabling the following hashes which are outdated and should not be used anymore.

1. From Notepad.exe, create a text file named **DisableHashes.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Hashes\MD5]
    "Enabled"=dword:00000000
    ```
3. Save **DisableHashes.reg**
4. Double-click the **DisableHashes.reg** file
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Configure cipher suites on Windows Server 2016

It is possible to configure the cipher suites with the help of a Group Policy Object (GPO). We can't configure them manually via ``Enable/Disable-TLSCipherSuite`` cmdlet if they were already configured via GPO. You can use the following PowerShell command to check if any cipher suites are configured via GPO:

```powershell
$cipherSuiteKeyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Cryptography\Configuration\SSL\00010002"  
if (((Get-ItemProperty $cipherSuiteKeyPath).Functions).Count -ge 1) {
	Write-Host "Cipher suites are configured by Group Policy" -Foregroundcolor Red
} else {
    Write-Host "No cipher suites are configured by Group Policy - you can continue with the next steps" -Foregroundcolor Green    
}
```

Configuring TLS 1.2 cipher suites on Windows Server 2016, is a 2-step task. The first task is to disable all existing cipher suites. This can be done via PowerShell:

1. Right click PowerShell and select _Run as administrator_
2. Copy and paste the following text into the elevated PowerShell window
    ```powershell
    foreach ($suite in (Get-TLSCipherSuite).Name) {
        if (-not([string]::IsNullOrWhiteSpace($suite))) {
            Disable-TlsCipherSuite -Name $suite -ErrorAction SilentlyContinue
        }
    }
    ```
3. Press _Enter_ and wait until execution has finished

The second task is to only enable the TLS 1.2 cipher suites. This can be done via PowerShell as well:

1. Right click PowerShell and select _Run as administrator_
2. Copy and paste the following text into the elevated PowerShell window
    ```powershell
    $cipherSuites = @('TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384',
                    'TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256',
                    'TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384',
                    'TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256',
                    'TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384',
                    'TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256',
                    'TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384',
                    'TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256')

    $suiteCount = 0
    foreach ($suite in $cipherSuites) {
        Enable-TlsCipherSuite -Name $suite -Position $suiteCount
        $suiteCount++
    }
    ```
3. Press _Enter_ and wait until the execution has finished
4. Restart the machine for the changes to take effect

### Configure cipher suites on Windows Server 2012 and Windows Server 2012 R2

1. From Notepad.exe, create a text file named **CipherSuitesOrder.reg**
2. Copy and paste the following text into the file
    ```notepad
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Cryptography\Configuration\Local\SSL\00010002]
    "Functions"="TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P384,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384_P384,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384_P256,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256_P384,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256_P256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA_P384,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA_P256,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA_P384,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA_P256,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256"
    ```
3. Save the **CipherSuitesOrder.reg**
4. Double click the **CipherSuitesOrder.reg**
5. Click _Yes_ to update your Windows Registry with these changes
6. Restart the machine for the changes to take effect

### Configure cipher curves (Windows Server 2016 only)

On Windows Server 2016 it is required to configure the elliptic curve preference. This can be done with an elevated PowerShell:

1. Right click PowerShell and select _Run as administrator_
2. Copy and paste the following commands to the elevated PowerShell and execute them one by one
    ```powershell
    Disable-TlsEccCurve -Name "curve25519"
    Enable-TlsEccCurve -Name "NistP384" -Position 0
    Enable-TlsEccCurve -Name "NistP256" -Position 1
    ```
3. Restart the machine for the changes to take effect
