# OU Membership Change
This query looks for OU changes within your Active Directory invironment. Depending on the environment and how strict you are, this could be a great custom alert. 

<br>

## Query 

```kql
SecurityEvent
| where EventID == 5139 //OU member change
| extend ActingUser = substring(substring(split(EventData, "<Data Name")[4], 19), 0, strlen(substring(split(EventData, "<Data Name")[4], 19)) - 11) // Depending on how your logs appear, you may need to tweak this line.
| extend OldOU = split(split(split(EventData, "<Data Name")[9], ">")[1], "<")[0]
| extend NewOU = split(split(split(EventData, "<Data Name")[10], ">")[1], "<")[0]
| extend ObjectClass = split(split(split(EventData, "<Data Name")[12], ">")[1], "<")[0]
| project ActingUser,OldOU,NewOU,ObjectClass
```

<br>

> [!CAUTION]
> If you have issues with the admin who made the change, you may need to tweak the query to show the user. <br>
> Expand the Event Data column and you can filter using the above methods. 
