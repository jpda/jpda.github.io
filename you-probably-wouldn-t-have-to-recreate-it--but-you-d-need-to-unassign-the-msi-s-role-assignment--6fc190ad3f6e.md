---
title: "You probably wouldn’t have to recreate it, but you’d need to unassign the MSI’s role assignment."
date: 2019-03-01T03:54:31.653Z
author: "John Patrick Dandison"

---

You probably wouldn’t have to recreate it, but you’d need to unassign the MSI’s role assignment. You can find the specific assignment with Get-AzureADServiceAppRoleAssignment and remove it with Remove-AzureADServiceAppRoleAssignment
