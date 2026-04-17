# OU Being Created
This query looks for OUs being created within your environment. Typically these should be changes that are infrequent and planned in advanced depending on your environment. These are good alerts to verify with your sysadmins.

## Query
``` kql
SecurityEvent
| where EventID == 5137 //OU Being Created
| extend Account = split(split(split(EventData, "<Data Name")[4], ">")[1], "<")[0]
| extend OUName = split(substring(split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0], 3), ",")[0]
| extend ObjectPath = split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0]
| extend ObjectClass = split(split(split(EventData, "<Data Name")[11], ">")[1], "<")[0]
| where ObjectClass == "organizationalUnit"
| project TimeGenerated, Account, OUName, ObjectPath, ObjectClass
```
