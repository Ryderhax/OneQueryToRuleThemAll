# Multi Admin Approval Process 
This query was built to assist with the MAA process. Specifically since intune does not notify the approver group (dumb) that there is an approval awaiting to be reviewed, this fills that gap and is designed to be paired with a logic app that will send a notification to a teams channel. 

<br>

## Approval Request Created 

``` kql
IntuneAuditLogs
| where OperationName == "Create ApprovalRequest"
| extend AdditionalDetails_ = tostring(parse_json(Properties).AdditionalDetails)
| extend CleanJustification = trim(" ", tostring(split(AdditionalDetails_, "=")[2]))
| extend Name_ = tostring(parse_json(tostring(parse_json(Properties).Targets))[0].Name)
| project TimeGenerated, Requestor = Identity, Action = Name_, Justification = CleanJustification
```

## Approval Request Approved 

``` kql
IntuneAuditLogs
| where OperationName == "Request Approved ApprovalRequest"
| extend ApprovalJustification = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).Targets))[0].ModifiedProperties))[1].New) //Approval Justification
| extend ApprovedTime = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).Targets))[0].ModifiedProperties))[0].New) //Approved Time and Date
| extend Aprover = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).Targets))[0].ModifiedProperties))[4].New)
| project TimeGenerated, Approver = Identity, ApprovedTime, ApprovalJustification
```
> [!NOTE]
> I am working on showing the request that was created previously to assist with visibility on what item was approved. At the moment, my use case doesn't call for it.



## Approval Request Denied 

``` kql

```
