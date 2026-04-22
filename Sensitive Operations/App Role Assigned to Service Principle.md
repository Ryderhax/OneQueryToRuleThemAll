# App Role Assigned to a Service Principle 
Tracks application permission changes to detect over‑permissioned service principals, privilege escalation, and OAuth‑based persistence in Entra ID. Depending on your organization this action reguarless of the permission granted, should be something the security team is aware of.
<br>

## Query

```kql

AuditLogs
| where OperationName == "Add app role assignment to service principal"
| extend Actor = tostring(parse_json(tostring(InitiatedBy.user)).userPrincipalName) //person granting the role
| extend ServicePrincipalDisplayName = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[6].newValue))) 
| extend AppPermission = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[1].newValue)))
| extend PermissionDesc = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[2].newValue)))
| extend ApplicationID = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[7].newValue)))
| extend ObjectID = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[5].newValue))) // SRVC Principle ObjectID
| extend AppRoleID = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[0].newValue)))
| project TimeGenerated, Actor, ServicePrincipalDisplayName, AppPermission, PermissionDesc, ApplicationID, ObjectID, AppRoleID

```
<br>

>[!NOTE]
> Line 6 - 9 are really just helpful items incase there is a correlation issue with the service principle display name
