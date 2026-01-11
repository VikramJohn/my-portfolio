---
status: approved
summary: "Modified csapi queries to include ID in where inputs"
# Hugo metadata
title: "[XYZ-767] added id in common resolvers"
date: 2025-12-15
version: "unreleased"

# MR tracking
mr_id: 
mr_url: 
owner: 

# Release note classification
jira_ticket: ["XYZ-767"]
changetype: "Changed"
component: ["Component Y"]
affected_services: ["authproc", "csapi"]

# Compatibility and deployment
backward_compatibility: "Incompatible"
services_to_stop: ["None"]
data_model_changes: "None"

# Additional tracking
client: ""
---
The `id` field was added to the `where` arguments of status-related queries in the **csapi** service. Specify the DB record ID---the `id` value in the corresponding DB table---in your query to get details about a specific status, substatus, substatus type or substatus attribute.

| Query | DB table to specify `id` value from 
| --- | --- 
| statuses | csdata.statuses |
| subStatuses | csdata.substatuses
| subStatusTypes | csdata.substatustypes
| subStatusAttributes | csdata.substatustypeattributes

## Upgrade notes

Update integrations that use the queries mentioned above.
