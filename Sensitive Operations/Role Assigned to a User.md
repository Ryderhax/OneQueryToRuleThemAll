# Role Assigned Directly to a User 
This query is great for finding any roles being directly assigned to a user either through PIM or by an Admin. <br>
<br>

## Query
```kql
AuditLogs
| where OperationName == "Add member to role"
| extend Actor = coalesce(
    tostring(InitiatedBy.user.userPrincipalName),
    tostring(InitiatedBy.app.displayName)
)
| extend Role = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[1].newValue)))
| extend Type = tostring(TargetResources[0].type)
| extend DisplayName = coalesce(
    tostring(TargetResources[0].displayName),
    tostring(TargetResources[0].userPrincipalName)
)
| extend RoleTemplateID = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[2].newValue)))
| project TimeGenerated, Actor, Type, Role, DisplayName, Result, RoleTemplateID
```
<br>

>[!NOTE]
> If you are using PIM you can remove the MS-PIM actor from the query, it can add some noise.
