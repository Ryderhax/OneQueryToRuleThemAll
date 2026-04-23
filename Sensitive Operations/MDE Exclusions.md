# Exclusions in Microsoft Defender for Eendpoint
This query looks for exclusions being put in locally on the machine. I would highly recommend that if you do not disable the ability for local admins to create exclusions on your workstations or servers, you use this as an alert. <br><br>

The KQL is going to show you events that have happened within your log retention window, I have included a powershell script for you to run that will return all exclusions within your environment (depending on how you run it) see that section for more details.
<br>

# Query
I have left a broad TimeGenerated line in the queries just for the sake of convienence, if you are running or looking into this for the first time it is helpful to understand times on when the exclusions were placed.

## Simple Exlusion Search
```kql
DeviceRegistryEvents
    | where TimeGenerated >ago(180d) 
    | where RegistryKey has @"\SOFTWARE\Microsoft\Windows Defender\Exclusions\"
    | project TimeGenerated, DeviceName, RegistryValueName, DeviceId;
```
<br>

## Add Last-Logon to Results (Workstations Only)
```kql
let exclusions =
    DeviceRegistryEvents
    | where TimeGenerated >ago(180d)
    | where RegistryKey has @"\SOFTWARE\Microsoft\Windows Defender\Exclusions\"
    | project TimeGenerated, DeviceName, RegistryValueName, DeviceId;
 
let lastlogon =
    DeviceLogonEvents
    | where LogonType in ("Interactive", "RemoteInteractive", "Unlock", "CachedInteractive")
    | where AccountName !in ("SYSTEM", "LOCAL SERVICE", "NETWORK SERVICE")
    | summarize arg_max(Timestamp, AccountName) by DeviceId
    | project DeviceId, LastLoggedOnUser = AccountName;
 
exclusions
| join kind=leftouter lastlogon on DeviceId
| project TimeGenerated, DeviceName, RegistryValueName, LastLoggedOnUser
| order by TimeGenerated desc

```
>[!WARNING]
>Do not use this for servers!\
>This script shows last logged on users not users who are responsible for the exclusions.
>Typically this works well for workstations since there should only be one user attributed to the machine. For servers, you will recieve anyone who was last logged on.

<br>

 ## Exclusions on Servers 
 >[!NOTE]
>Currently WIP

<br>

## Powershell Script
This was made for the specific purpose of returning exclusions set on workstations througout the organization. Using the scripts and remeditions section within intune, this lets us get all the workstations, thier users or owners, the exclusions set on the machine, and then export them via JSON to put in a single CSV for export. If you were only looking for a single machine either server or workstation you can use the _"Defender exclusions"_ section of the script below.

```powershell
$ErrorActionPreference = 'Stop'

function To-Array {
    param([object]$Value)
    if ($null -eq $Value) { return @() }
    if ($Value -is [Array]) { return $Value }
    return @($Value)
}

# Last logged-on user
try {
    $lastUser = (Get-CimInstance Win32_ComputerSystem).UserName
    if ([string]::IsNullOrWhiteSpace($lastUser)) { $lastUser = "Unknown" }
} catch {
    $lastUser = "Unknown"
}

# Defender exclusions
try {
    $mp = Get-MpPreference
} catch {
    $mp = $null
}

$result = [PSCustomObject]@{
    DeviceName         = $env:COMPUTERNAME
    LastLoggedOnUser   = $lastUser
    ExclusionPath      = if ($mp) { To-Array $mp.ExclusionPath }      else { @() }
    ExclusionProcess   = if ($mp) { To-Array $mp.ExclusionProcess }   else { @() }
    ExclusionExtension = if ($mp) { To-Array $mp.ExclusionExtension } else { @() }
}

# Output to JSON for CSV
Write-Output ($result | ConvertTo-Json -Depth 6 -Compress)
exit 0

```
