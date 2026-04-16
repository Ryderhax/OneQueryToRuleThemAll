# Security Event Logs
This query looks for the successful or failed login events _(4624 & 4625)_ via the Windows Security Event Logs. Useful when looking for server logons. 

<br>

## Query 

```kql
SecurityEvent
| where TimeGenerated >ago(30d)
| where EventID == 4624 //Successful logon event ID
//| where EventID == 4625 //Failed logon event ID
| extend LogonTypeName = case(
    LogonType == 2, "Interactive (Local Console)",
    LogonType == 3, "Network (Share Access)",
    LogonType == 4, "Batch (Scheduled Task)",
    LogonType == 5, "Service (Service Account)",
    LogonType == 7, "Unlock (Workstation Unlock)",
    LogonType == 8, "NetworkCleartext (Cleartext Auth)",
    LogonType == 9, "NewCredentials (RunAs)",
    LogonType == 10, "RemoteInteractive (RDP)",
    LogonType == 11, "CachedInteractive (Offline Login)",
    LogonType == 12, "CachedRemoteInteractive (Cached RDP)",
    LogonType == 13, "CachedUnlock (Unlock with Cached Creds)",
    "Other"
)
| extend LogonResult = case(
    EventID == 4624, "An account was successfully logged on.",
    EventID == 4625, "An account failed to log on.",
    "idk what happened bruh"
)
| project TimeGenerated, SourceHost = WorkstationName, AuthenticatedAccount = Account, LogonResult, DestinationHost = Computer, IpAddress, IpPort, LogonTypeName
```

<br>

>[!NOTE]
> _I don't normally need the entire output so I filter line 23 based off what I commonly use and look at. Remove lin 23 if you want to see authentication info._
