---
layout: post
title: How the versioning works in SharePoint Framework solutions
date: '2018-10-31T10:42:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Client Side Web Part
- Client Web Part
- package-solution
- SharePoint Framework
- SharePoint Framework Extensions
- version
- App Catalog
- Feature
- SharePoint
modified_time: '2018-11-06T15:53:11.702-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1748601231724872158
blogger_orig_url: http://blog.aterentiev.com/2018/10/how-versioning-works-in-sharepoint.html
---

This post is a sum up of the <a target="_blank" href="https://twitter.com/alexaterentiev/status/1057389388588441600">conversation</a> held in Twitter regarding versioning of your SharePoint Framework solutions.<br />In <span class="style">package-solution.json</span> file of the SPFx project there is a property <span class="code">version</span><br />The description from the <a target="_blank" href="https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/basics/notes-on-solution-packaging"></a> currently tells us that:<br /><blockquote>Optionally, you may also specify a version number in the format "X.X.X.X", which is used to identify various versions of the package when upgrading. </blockquote><br />But it doesn't explain what is actually affected by the version update.<br />So, here is how it actually works:<br /><ul><li>First of all, all the assets (JavaScript files, CSS, images, etc.) are automatically updated as soon as new version has been added to the App Catalog. You don't need to update the app on all of your sites. </li><li>If you're using Feature Framework and Feature declarations in your solution then these are the resources that need the actual App update. </li></ul>This situation may lead to confusion but it is what it is.<br />And actually it makes sense for two reasons: <ol><li>As an app should be installed to every site (meaning every root site and if needed - every sub site) separately, it's really difficult to go through all the sites and update the app. So, automatic update is a good decision.</li><li>Why this "automatic" update isn't applicable to Feature Framework? Because new version of features content (like fields, content types, etc.) could contain breaking changes and it's better to be aware of it. And anyway SharePoint Dev team does not recommend to use Feature Framework in your solutions at all. =)</li></ol>Here is also a <a href="https://youtu.be/dxD0lgYUCI4" target="_blank">PnP Shorts video</a> that describes how to update the SPFx solution.<br />Hopefully, this information will be helpful.<br /><br />Have fun!  