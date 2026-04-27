# Incidents in the Last 30 days
This query looks for all security alerts within the last 30 days. I have de-dupped the incidents so you should be seeing the latest comments/updates from that incident. I recommend you to verify that this is the case.

## Query
```kql
SecurityIncident 
| summarize arg_max(TimeGenerated, *) by IncidentNumber 
| where CreatedTime >= ago(30d) and CreatedTime <= now()
| extend userPrincipalName_ = tostring(Owner.userPrincipalName) 
| extend Product = todynamic((parse_json(tostring(AdditionalData.alertProductNames))[0]))
| extend Owner = todynamic(Owner.assignedTo) 
| project
    IncidentNumber,
    Created = format_datetime(TimeGenerated, 'MM-dd-yy'),
    Title,
    Severity,
    CreatedTime,
    ClosedTime,
    Status,
    Classification,
    ClassificationReason,
    ClassificationComment
| order by Created asc
```

