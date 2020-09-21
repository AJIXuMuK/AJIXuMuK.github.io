---
layout: post
title: Migrating Power Apps SharePoint List Form
description: Step-by-step instruction on how to migrate custom Power Apps SharePoint list form to another list
slug: migrating-power-apps-sharepoint-list-form
date: 2020-09-20T18:36:24.391Z
tags:
    - Power Apps
    - PowerApps
    - M365
    - Microsoft
    - Microsoft 365
    - Migration
    - O365
    - Office 365
    - SharePoint
    - SharePoint Online
    - Forms
    - PowerPlatform
    - Power Platform
featured_image: assets/images/posts/2020/2020-09-20-featured.png
featured_image_thumbnail: assets/images/posts/2020/2020-09-20-featured.png
---

When you customize SharePoint List Form using Power Apps you would probably prefer to complete it on test environment and them migrate or export to production. Unfortunately, currently there is no direct point-and-click way to do that.
In this post I want to describe the steps to export your custom form, even if you have lookup columns or additional SharePoint connections.
[Tomasz Poszytek](https://twitter.com/TomaszPoszytek) has a [great post](https://poszytek.eu/en/microsoft-en/office-365-en/powerapps-en/importing-powerapps-package-as-a-sharepoint-list-form/) how to do the export/import. And these steps are enough if you don't have additional SharePoint connections inside your app, or if you don't want to rename the data sources.
If you do have either of these requirements, you'll need to proceed with additional steps.
But let's start from the beginning.

## Export Power Apps App
After your custom form is ready, you'll need to export it.
It is done in Power Apps Studio: go to **File -> Save** and click on **See all versions**.
![See All Versions](./assets/images/posts/2020/2020-09-20-see-all-versions.png)<br /><br />
Click **Export package** in the command bar on the App Details page:
![Export package](./assets/images/posts/2020/2020-09-20-export-app.png)<br /><br />
Now, enter Name of the package and select **Import Setup** configuration: either Create as New, or Update. Click **Export**. It will generate a `.zip` file downloaded to your machine.
![Export](./assets/images/posts/2020/2020-09-20-export-package.png)

## Modifying the Package
Next step is to modify the package (generated `.zip` archive) to update SharePoint connections.
### Updating Connection References and Embedded App Info
First thing to do is to modify `connectionReferences` and `embeddedApp` sections of the app.<br />Open `.zip` and navigate to *Microsoft.PowerApps/apps/&lt;numeric-value&gt;* folder. There you'll find the only `.json` file.
![JSON File](./assets/images/posts/2020/2020-09-20-json.png)<br /><br />
Open it and find `connectionReferences` section. It contains all the connections in your app, including SharePoint lists connections, and others (for example, Azure AD connection, or Outlook connection).<br />
Look for connections with `"displayName": "SharePoint"`, and modify it for your needs:
- To update list connection specify list GUID as `tableName` and site url as section name in `dataSets` object
- To update the data connection name update the names in `dataSources` sections of the connection


![Connections](./assets/images/posts/2020/2020-09-20-data-sets.png)<br /><br />

Now, search for `embeddedApp` section and update `siteId`, `listId`, and `listUrl` with new site Url, list GUID (similarly to `dataSets`) and list URL.<br />
![Embedded App](./assets/images/posts/2020/2020-09-20-embedded-app.png)<br />

### Updating Data Sources References in the Components
Now, when the data source have been updated, we need to iterate through the app's components and update references to the data sources.<br />
In the same folder as `.json` file locate `.msapp` file.<br /> 
![MS App File](./assets/images/posts/2020/2020-09-20-msapp.png)<br /><br />
It is also an archive. So, if you rename it to `.zip` you'll be able to open it as a regular `.zip` archive.<br />
Inside you'll find a set of `.json` files.<br />
![MS App Content](./assets/images/posts/2020/2020-09-20-msapp-content.png)<br /><br />
First, you need to open `References\DataSources.json` file, search for all "old" list GUIDs, URLs, and data sources' name, and replace them with the new ones - same as `connectionReferences` and `embeddedApp` sections.<br />
![Replace GUID](./assets/images/posts/2020/2020-09-20-replace.png)<br /><br />
Second, if you renamed the data source(s) you need to iterage through all `Controls\*.json` files and update the names of the data source(s).<br />
![Controls](./assets/images/posts/2020/2020-09-20-controls.png)<br /><br />
When all the chages are done, you need to rename `.zip` back to `.msapp`.

## Importing the Package/App
Now you can navigate to Power Apps portal and click **Import canvas app** on **Apps** tab and select the `.zip` package from the import (that now contains all the modifications):<br />
![Import](./assets/images/posts/2020/2020-09-20-import.png)<br /><br />
If there are any errors during the import, go back and check that all URLs and GUIDs are correct.<br />
If everything is done correctly, you'll the custom form for the list.<br />
I would also recommend to Edit the from from list (Power Apps -> Customize forms) and check that all the connections are there and all the cards/controls use correct references.

<br />
That's all for today!<br />
Have fun!
