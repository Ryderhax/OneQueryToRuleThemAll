# User Added to a Group
This query looks for **_all users_** who are added to a specific group and by who. The origional use case for this query was to look for users who are being added to sensitive groups like role assignable groups and by who. This is extremely helpful for organizations who utilize role assignable groups for job functions or who provides access to certain applications or exclusions based on groups. 

<br>

## Query 

```kql
AuditLogs
| where TimeGenerated >ago(30d)
| where ActivityDisplayName == "Add member to group"
| extend Admin = tostring(parse_json(tostring(InitiatedBy.user)).userPrincipalName) //this is the person who added them to the group. 
| extend User_Added = tostring(TargetResources[0].userPrincipalName) //this is going to be the person who is being added to the group.
| extend Group = tostring(parse_json(tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[1].newValue))) //group the user is being added to.
//| where User_Added contains "SPECIFIC_USER" //you only need this if you are looking for a specific user. 
| where Group contains "YOUR-GROUP" 
| project TimeGenerated, Admin, User_Added, Group //filtering out the other noise not really wanted for the query. you can remove this to see more information.
```

<br>

> [!note]
> Line 7 is only needed if you are monitoring a <ins>specific user</ins> whose been added to a group. <br>
> Line 8 can be removed if you want to see whose been added to <ins>any group.</ins>
