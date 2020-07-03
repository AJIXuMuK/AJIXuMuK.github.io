---
layout: post
title: 'Workaround. SharePoint App Catalog: Sync to Teams button doesn''t work.'
date: '2019-03-29T17:04:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Sync to Teams
- SPFx
- Microsoft Teams
- O365
- SharePoint Framework
- Office 365
- App Catalog
- SharePoint
modified_time: '2019-03-29T17:06:08.635-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-7555041371177713138
blogger_orig_url: http://blog.aterentiev.com/2019/03/workaround-sharepoint-app-catalog-sync.html
---

Currently "Sync to Teams" button in SharePoint App Catalog doesn't work or works unstable.<br />Here is a <a href="https://github.com/SharePoint/sp-dev-docs/issues/3619" target="_blank">related issue</a><br />As a result, if you create a new SPFx project based on v1.8 you will not be able to add it to Teams organizational store.<br />You will still be able to do that if you create your own Team's app manifest (it was generated by Yeoman SPFx generator v1.7.1) and create a .zip archive to contain this manifest and icons from <span class="code">teams</span> folder.<br />As the manifest file is no longer generated I decided to provide its content here. <br /><a name='more'></a><br />
<div markdown="1">
{% highlight javascript %}
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.2/MicrosoftTeams.schema.json",
  "manifestVersion": "1.2",
  "packageName": "YOUR_PACKAGE_NAME",
  "id": "YOUR_WEB_PART_ID",
  "version": "0.1",
  "developer": {
    "name": "Dev",
    "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
    "privacyUrl": "https://privacy.microsoft.com/en-us/privacystatement",
    "termsOfUseUrl": "https://www.microsoft.com/en-us/servicesagreement"
  },
  "name": {
    "short": "TEAMS_APP_NAME"
  },
  "description": {
    "short": "TEAMS_APP_DESCRIPTION",
    "full": "TEAMS_APP_DESCRIPTION"
  },
  "icons": {
    "outline": "YOUR_WEB_PART_ID_outline.png",
    "color": "YOUR_WEB_PART_ID_color.png"
  },
  "accentColor": "#004578",
  "configurableTabs": [
    {
      "configurationUrl": "https://{teamSiteDomain}{teamSitePath}/_layouts/15/TeamsLogon.aspx?SPFX=true&amp;dest={teamSitePath}/_layouts/15/teamshostedapp.aspx%3FopenPropertyPane=true%26teams%26componentId=YOUR_WEB_PART_ID",
      "canUpdateConfiguration": false,
      "scopes": [
        "team"
      ]
    }
  ],
  "validDomains": [
    "*.login.microsoftonline.com",
    "*.sharepoint.com",
    "*.sharepoint-df.com",
    "spoppe-a.akamaihd.net",
    "spoprod-a.akamaihd.net",
    "resourceseng.blob.core.windows.net",
    "msft.spoppe.com"
  ],
  "webApplicationInfo": {
    "resource": "https://{teamSiteDomain}",
    "id": "00000003-0000-0ff1-ce00-000000000000"
  }
}

{% endhighlight %}
</div>
You can take this JSON, replace <span class="code">YOUR_PACKAGE_NAME,YOUR_WEB_PART_ID,TEAMS_APP_NAME,TEAMS_APP_DESCRIPTION</span>, save as <span class="code">manifest.json</span> in <span class="code">teams</span> folder, create .zip archive that contains <span class="code">teams</span> content and upload to Teams Organizational Store.<br />One note: do not create .zip archive in <span class="code">teams</span> folder as it leads to issues when adding SPFx solution to the App Catalog.<br /><br />That's all for today!<br />Have fun! 