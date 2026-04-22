# Role Assigned to Service Principle
This query looks for any service principle that is given a built-in role. This is similar to the [Role Assigned to a User]() query.

<br>

```kql
AuditLogs
| where OperationName == "Add member to role"
| extend Actor = coalesce(
    tostring(InitiatedBy.user.userPrincipalName),
    tostring(InitiatedBy.app.displayName)
)
| extend AssignedRole = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[1].newValue)))
| extend Type = tostring(TargetResources[0].type)
| extend DisplayName = coalesce(
    tostring(TargetResources[0].displayName),
    tostring(TargetResources[0].userPrincipalName)
)
| extend RoleTemplateID = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[2].newValue)))
| where Type == "ServicePrincipal"
| project TimeGenerated, Actor, Type, AssignedRole, DisplayName, Result, RoleTemplateID
```
<br>

>[!NOTE]
> If you are using PIM you can remove the MS-PIM actor from the query, it can add some noise.
