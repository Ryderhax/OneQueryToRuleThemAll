# OU Being Modified
This query looks for OUs being modified and what is being changed in that OU. 

<br>

> [!IMPORTANT]
> This is still under development. I am currenlty working on being able to see what is being changed. This will be updated eventually when I remember to comeback in here and change it.

<br>

## Query

``` kql
SecurityEvent
| where TimeGenerated >ago(180d)
| where EventID == 5136 //OU Being Modified
| extend Account = split(split(split(EventData, "<Data Name")[4], ">")[1], "<")[0]
| extend OUName = split(substring(split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0], 3), ",")[0]
| extend ObjectPath = split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0]
| extend ObjectClass = split(split(split(EventData, "<Data Name")[11], ">")[1], "<")[0]
| where ObjectClass == "organizationalUnit"
| project TimeGenerated, Account, OUName, ObjectPath, ObjectClass
```

