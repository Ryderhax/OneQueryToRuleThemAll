# Incidents by Detection Source
This query shows how many alerts per detection source you have, its best when shown for the last 7-30d. 

## Query
```kql
SecurityIncident
| summarize arg_max(TimeGenerated,CreatedTime,Status, Severity, AdditionalData, IncidentUrl, Comments, Classification,ClassificationReason, ClassificationComment,Labels, Title, AlertIds) by IncidentNumber
| extend Tactics = todynamic(AdditionalData.tactics)
| extend Product = todynamic((parse_json(tostring(AdditionalData.alertProductNames))[0]))
| extend Tags = extract_all('labelName":"(.*?)"',tostring(Labels))
| extend Products = strcat_array(AdditionalData.alertProductNames, ", "), Alerts = tostring(AdditionalData.alertsCount), Bookmarks = tostring(AdditionalData.bookmarksCount), Comments = tostring(AdditionalData.commentsCount), Tactics = strcat_array(AdditionalData.tactics, ", "), Labels = strcat_array(Tags, ", ")
| mvexpand AlertIds to typeof(string)
| join kind=leftouter
(SecurityAlert
| summarize arg_max(TimeGenerated,AlertName, Description, AlertType, Entities)by SystemAlertId) on $left.AlertIds == $right.SystemAlertId
| summarize AlertName = makelist(AlertName), AlertType = makelist(AlertType) by Comments, Labels, Title, Products, AlertsCount = Alerts, Bookmarks, Status, Severity, TimeGenerated, ClassificationComment, Classification, ClassificationReason 
| extend AlertNames = strcat_array(AlertName, ", "), AlertTypes = strcat_array(AlertType, ", ")
| summarize count() by bin(TimeGenerated, 1d), Products
| render columnchart 
```
