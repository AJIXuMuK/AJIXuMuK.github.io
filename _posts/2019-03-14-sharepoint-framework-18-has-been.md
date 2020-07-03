---
layout: post
title: SharePoint Framework 1.8 Has Been Released!
date: '2019-03-14T12:57:00.001-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SPFx
- Office Development
- Microsoft Teams
- Teams
- O365
- SharePoint Framework
- Office 365
- SharePoint Development
- SharePoint
modified_time: '2019-03-15T15:34:41.050-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-8655882614578171648
blogger_orig_url: http://blog.aterentiev.com/2019/03/sharepoint-framework-18-has-been.html
---

Today the SharePoint Framework 1.8 has been released!<br />Why is that important and excited? Because this version makes SPFx a little bit more than just a framework for SharePoint development. Because now SPFx can be used to implement Microsoft Teams configurable Tabs!<br />This functionality was in preview since v 1.7.0 but now it's GA-d with some changes in comparison with preview version:<br /><br /><ol><li>You can configure if the web part is available as a standard web part, as a tab in Teams, or as an "App part". This configuration is done in web part's manifest file using <span class="code">supportedHost</span> property:<br />
<div markdown="1">
{% highlight javascript %}

{
  "supportedHosts": ["SharePointWebPart", "TeamsTab", "SharePointFullPage"]
}

{% endhighlight %}
</div>
</li><li>Now developer doesn't need to manually create .zip archive and deploy it to Teams app catalog. This step is automated using "Sync to Teams" ribbon button in SharePoint App Catalog:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/sync-to-teams.png" /></li><li>Web part's Property Pane can be launched at any time from Team Tab's menu.<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2019/teams property pane.png" /></li></ol>Besides of SPFx in Teams there are other cool new features and improvements: <ul><li>App part pages - new page layout with locked UI that can contain single web part. Basically, these are SPAs... in SharePoint way. This is configured for the web part using <span class="code">supportedHosts</span> property mentioned above</li><li>Support of TypeScript 2.x and 3.x. Until now SPFx supported TypeScript 2.x only. But now, if needed, developers can use TypeScript 3.x with such new features as project references, tuples in REST parameters, <span class="code">unknown</span> type, and others. You can read more about TypeScript release notes in <a href="https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html" target="_blank">TypeScript handbook</a></li><li>Web part variant themeing in SharePoint modern pages - now web parts can use <span class="code">ThemeProvider</span> and render their background based on theme and section's color instead of default white background. Use <span class="code">"supportsThemeVariants": true</span> in web part's manifest to enable this feature</li><li>Library Components (Preview) - ability to create components' libraries to be referenced by other SPFx projects</li><li>.. other improvements that are <b>subjectively</b> less important</li></ul>Here are some helpful links about SPFx v1.8: <ul><li><a href="https://developer.microsoft.com/en-us/sharepoint/blogs/announcing-the-general-availability-of-sharepoint-framework-1-8/" target="_blank">https://developer.microsoft.com/en-us/sharepoint/blogs/announcing-the-general-availability-of-sharepoint-framework-1-8/</a> - announcement <li><a href="https://github.com/SharePoint/sp-dev-docs/wiki/SharePoint-Framework-v1.8-release-notes" target="_blank">https://github.com/SharePoint/sp-dev-docs/wiki/SharePoint-Framework-v1.8-release-notes</a> - Release notes</li><li><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/integrate-with-teams-introduction" target="_blank">https://docs.microsoft.com/en-us/sharepoint/dev/spfx/integrate-with-teams-introduction</a>Introduction to SPFx in Microsoft Teams</li><li><a href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/using-web-part-as-ms-teams-tab" target="_blank">https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/using-web-part-as-ms-teams-tab</a>Tutorial on how to build Teams Tab using SPFx</li></ul>Enjoy new round of improvements from SharePoint Dev team!<br /><br />That's all for today!<br />Have fun! 