#Powershelling

Using the powershell on the computer 
- install the exchangeonlinemanagement 

```powershell
install-module exchangeonlinemanagement
```

### Prerequest


Running in Powershell to enable the TLS1.2
```powershell
[Net.ServicePointManager]::SecurityProtocol =
    [Net.ServicePointManager]::SecurityProtocol -bor
    [Net.SecurityProtocolType]::Tls12
```

Get the Version of the Powershell Get
```
Get-Module PowerShellGet, PackageManagement -ListAvailable
```

```
Set-ExecutionPolicy Unrestricted 
Install-PackageProvider NuGet -Force

# This install the ExchangeOnlineManagement

Install-Module -Name ExchangeOnlineManagement

```

Now connecting to it with powershell
```
connect-exchange 
```
You need to login

```
disconnect-exchange
```
