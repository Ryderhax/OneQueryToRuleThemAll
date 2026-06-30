# Purview eDiscovery Case Creation 
This query is to assist with notifying teams of a case creation within purview. Without native notifications this is a nice query you can turn into a scheduled alert. 

If you are not already, I would recoomend auditing case managers and ediscovery managers.


## Query

```kql
CloudAppEvents
| where TimeGenerated >ago(180d)
| where ActionType == "CaseAdded"
| extend CaseName = tostring(RawEventData.CaseName)
| extend Case_ = tostring(RawEventData.Case)
| extend Case = coalesce(CaseName, Case_)
| project TimeGenerated, Creator = AccountDisplayName, Case
```
<br>

>[!NOTE]
> Line 5 and 6 are combined to try to combine both case and case name.
