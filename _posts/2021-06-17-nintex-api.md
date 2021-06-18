---
layout: post
title: Working with Nintex REST API for Office 365
description: Personal experience of working with Nintex API to import & export workflows and forms
date: 2021-06-18T02:29:09.583Z
slug: working-nintex-rest-api-office-365
featured_image: assets/images/posts/2021/2021-06-17-nintex.png
featured_image_thumbnail: assets/images/posts/2021/2021-06-17-nintex.png
tags:
  - M365
  - Microsoft 365
  - O365
  - Office 365
  - SharePoint
  - SharePoint Online
  - Workflow
  - Nintex
  - API
---
Nintex Workflows and Forms are well-known products used in hundreds (or thousands) organizations across the world. They allow to build custom SharePoint Forms for SharePoint On-Prem and Online (including responsive forms) and custom workflows with tons of actions. No wonder customers want to automate import & export or, in other words - provisioning of such assets.
In this post, I want to describe my personal experience of implementing an import/export engine using PowerShell and Nintex RESI API for Office 365.

## Overall Expression
In general, I can say the API is pretty simple and straightforward. 
There are few endpoints you need to know. And all of them are described in the [documentation](https://help.nintex.com/en-us/sdks/sdko365/Default.htm) with required parameters and examples.
The API is much easier than Microsoft Power Automate. There is no need to manually modify anything in the packages when importing a form or a workflow to the destination (in Power Automate, you'll need to update data sources' connections before import).
The whole code of import/export for workflow is just a few lines:
```bash

# Export
$restURL = "https://$WorkflowAPIBaseUrl/api/v1/workflows/packages/$WorkflowId"  
$creds = New-Object -TypeName Microsoft.SharePoint.Client.SharePointOnlineCredentials -ArgumentList $Credential.UserName, $Credential.Password
$authCookie = $creds.GetAuthenticationCookie($SourceSiteUrl)  
$cookie = "cookie $SourceSiteUrl " + $authCookie  

$headersExport = @{  
    'Authorization' = $cookie;
    'Api-Key'       = $WorkflowAPIKey;
    'Accept'        = 'application/json';
    'Content-Type'  = 'application/nwp'
}  

Invoke-RestMethod -Method "Get" -Uri $restURL -Headers $headersExport -OutFile $WorkflowFilePath 

# Import
$restUrl = "https://$WorkflowAPIBaseUrl/api/v1/workflows/packages/?migrate=true&listTitle=$DestDocLibTitle"
$headersImport = @{  
    'Authorization' = $cookie
    'Api-Key'       = $WorkflowAPIKey
    'Accept'        = "application/json"  
}  

$response = Invoke-RestMethod -Method "POST" -Uri $restUrl -Headers $headersImport -InFile $WorkflowFilePath

# Publish
$restUrl = "https://$WorkflowAPIBaseUrl/api/v1/workflows/$WorkflowId/published"  
$headersPublish = @{  
    'Authorization' = $cookie;
    'Api-Key'       = $WorkflowAPIKey;
    'Accept'        = "application/json";  
}  


Invoke-RestMethod -Method "POST" -Uri $restUrl -Headers $headersPublish
```

Same for forms:
```bash

# Export
$creds = New-Object -TypeName Microsoft.SharePoint.Client.SharePointOnlineCredentials -ArgumentList $Credential.UserName, $Credential.Password
$restUrl = "https://$FormsAPIBaseUrl/api/v1/forms/$DocLibId,$ctId"

$authCookie = $creds.GetAuthenticationCookie($SourceSiteUrl)
$cookie = "cookie $SourceSiteUrl " + $authCookie
$headers = @{ 'Authorization' = $cookie; 'Api-Key' = $FormsAPIKey; 'Accept' = "application/json" }

Invoke-RestMethod -Method "GET" -Uri $restUrl -Headers $headers -OutFile $FormFilePath

# Import
$restUrl = "https://$FormsAPIBaseUrl/api/v1/forms/$DestDocLibId,$ctId"

$authCookie = $creds.GetAuthenticationCookie($DestSiteUrl)
$cookie = "cookie $DestSiteUrl " + $authCookie
$headers = @{ 'Authorization' = $cookie; 'Api-Key' = $FormsAPIKey; 'Accept' = "application/json" }

# Import
$response = Invoke-RestMethod -Method "POST" -Uri $restUrl -Headers $headers -InFile $FormFilePath

#Publish
$response = Invoke-RestMethod -Method "POST" -Uri "$restUrl/publish" -Headers $headers
```

As you can see, it's pretty easy to implement the automation of import and export for both workflows and forms.
But there are some caveats you should remember and consider when planning such automation or promising something to the customer.

## Caveats
### Request Nintex API Key
Nintex API Key is used in all requests and provided as an `Api-Key` header value.
The caveat related to the API Key is you need to request it by email. And even for large organizations, it can take a week or so to get it. So plan accordingly (request it in advance if possible) to avoid it being a blocker.

### The only available authentication method is an authentication cookie
Nintex API supports cookine authentication only. It means you can't use .NET Core and, as a result, PowerShell 7, the latest version of PnP PowerShell, etc.

### New Responsive Forms are not supported
Nintex has three options to design forms:
- Classic Forms
- Responsive Forms
- New Responsive Forms
The most current version is New Responsive Forms designer that provides the latest and greatest, including better support of responsive design, better rules formulas, etc.
But... but these forms **are not supported in API**. If you have such forms - either replace them with Responsive Forms or forget about API.
What's even worse - if you have Task Forms in a workflow created using New Responsive Forms designer, ** the whole workflow will not be imported**. And the only error you've got is 500 with no meaningful description.

### Shared Connections are not persisted
Recently, Nintex added a new feature to Workflows for Office 365 - [Connections](https://help.nintex.com/en-US/office365/Workflows/Connections.htm). Connections allow using OAuth authentication, which was not possible before that.
Connections are used for different types of actions. For example, Terminate Workflow.
There are two types of connections: Personal and Shared. Shared connections have three levels of availability:
- Site - connection is available for any workflow on the current site
- Site Collection - connection is available for any workflow on the current site collection
- Tenant - connection is available for any workflow on any site on the tenant
Intuitively you would expect the Shared connection, at least the Tenant connection, to be persisted when exporting and importing a workflow.
But that is not true. You will need to manually configure the connection on the destination. Manual step in the automatic process? Not good :)

### No easy way to change workflow name
In most cases, it's not an issue. But if you plan to have multiple instances of the same workflow on a single site - you need to change the name. Otherwise, all imported workflows, except the first one, will be set to Development mode until you rename and republish them.
If you try to import a workflow multiple times on the same site - you will not receive any errors. So this issue is a bit tricky to pinpoint.

## Conclusion
In many scenarios, Nintex REST API for Office 365 provides an easy way to implement automation for both forms and workflows used in SharePoint.
But there are some caveats that can become unresolvable blockers. Take them into consideration during the planning phase.


That's all for today!<br />
Have fun!
