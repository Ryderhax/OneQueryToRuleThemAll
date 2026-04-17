# OU Deletion
This query shows you events for an OU being deleted within your environment. Similar to the creation or modification, these should alrady be monitored and alerted on.

<br>

```kql
SecurityEvent
| where EventID == 5141 //OU Being Deleted
| extend Account = split(split(split(EventData, "<Data Name")[4], ">")[1], "<")[0]
| extend OUName = split(substring(split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0], 3), ",")[0]
| extend ObjectPath = split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0]
| extend ObjectClass = split(split(split(EventData, "<Data Name")[11], ">")[1], "<")[0]
| where ObjectClass == "organizationalUnit"
| project TimeGenerated, Account, OUName, ObjectPath, ObjectClass
```
