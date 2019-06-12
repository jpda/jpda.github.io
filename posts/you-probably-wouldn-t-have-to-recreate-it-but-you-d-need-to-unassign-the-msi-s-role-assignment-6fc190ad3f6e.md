---
title: >-
  You probably wouldn’t have to recreate it, but you’d need to unassign the
  MSI’s role assignment.
description: ''
date: '2019-03-01T03:54:31.653Z'
categories: []
keywords: []
slug: >-
  /@jpda/you-probably-wouldnt-have-to-recreate-it-but-you-d-need-to-unassign-the-msi-s-role-assignment-6fc190ad3f6e
---

You probably wouldn’t have to recreate it, but you’d need to unassign the MSI’s role assignment. You can find the specific assignment with Get-AzureADServiceAppRoleAssignment and remove it with Remove-AzureADServiceAppRoleAssignment