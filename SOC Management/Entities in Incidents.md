# Entities in Incidents
This query shows the frequency of entities appearing in incidents.

I did not make this one and am not sure who did. I do find this incredibly helpful and useful though so I have included it in here.

## Query
```kql
SecurityAlert
| extend Entities = iff(isempty(Entities), todynamic('[{"dummy" : ""}]'), todynamic(Entities))
| mvexpand Entities
| evaluate bag_unpack(Entities, "Entity_")
| extend Entity_Type = columnifexists("Entity_Type", "")
| extend Entity_Name = columnifexists("Entity_Name", "")
| extend Entity_ResourceId = columnifexists("Entity_ResourceId", "")
| extend Entity_Directory = columnifexists("Entity_Directory", "")
| extend Entity_Value = columnifexists("Entity_Value", "")
| extend Entity_HostName = columnifexists("Entity_HostName", "")
| extend Entity_Address = columnifexists("Entity_Address", "")
| extend Entity_ProcessId = columnifexists("Entity_ProcessId", "")
| extend Entity_Url = columnifexists("Entity_Url", "")
| extend Target = iif(Entity_Type == "account", Entity_Name, iif(Entity_Type == "azure-resource", Entity_ResourceId, iif(Entity_Type == "cloud-application", Entity_Name, iif(Entity_Type == "dns", Entity_Name, iif(Entity_Type == "file", strcat(Entity_Directory, "\\", Entity_Name), iif(Entity_Type == "filehash", Entity_Value, iif(Entity_Type == "host", Entity_HostName, iif(Entity_Type == "ip" , Entity_Address, iif(Entity_Type == "malware", Entity_HostName, iif(Entity_Type == "network-connection", Entity_Name, iif(Entity_Type == "process", Entity_ProcessId, iif(Entity_Type == "registry-key", Entity_Name, iif(Entity_Type == "registry-value", Entity_Name, iif(Entity_Type == "security-group", Entity_Name, iif(Entity_Type == "url", Entity_Url, "NoTarget")))))))))))))))
| where Entity_Type in ("account", "host", "ip", "url", "azure-resource", "cloud-application", "dns", "file", "filehash", "malware", "network-connection", "process", "registry-key", "registry-value", "security-group")
| summarize count() by bin(TimeGenerated, 1d), Target, Entity_Type
| project-away TimeGenerated
| order by count_ desc
```
