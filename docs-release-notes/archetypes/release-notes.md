---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
version: "" # Release version this belongs to

# MR tracking
mr_id: 
mr_url: 
owner: 

# Release Note Metadata
summary: "" # Brief one-line summary for list views
jira_ticket: "" # Jira ticket number 
upgrade_notes: false # true if special upgrade steps required

# Categorization
changetype: "" # added, changed, fixed, deprecated, compliance, api
components: [] # front, back, commonserver, etc.
affected_services: [] # List of affected services

# Impact Assessment
services_to_stop: false # or list of services [] -- add processing for this
data_model_changes: false # true if database/schema changes
backward_compatibility: false # true if this breaks backward compatibility

# layouts
type: page

# new -- try using layout per https://gohugo.io/templates/new-templatesystem-overview/#changes-to-template-lookup-order. Create a custom layout for release notes.
layout: release-note
---

## Description

Brief description of what changed and why.

## Upgrade Notes

Specific steps required for upgrading, if any.

