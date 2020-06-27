---
layout: post
title: 'Remember: Web GUID is NOT UNIQUE'
date: '2019-05-30T18:08:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SharePoint 2016
- Office 365
- SharePoint Development
- SharePoint On-Prem
- SharePoint
- SharePoint 2019
- SharePoint 2013
- Office Development
- Web
- Guid
- Unique Identifier
modified_time: '2019-06-03T17:19:32.188-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4230448755980875788
blogger_orig_url: http://blog.aterentiev.com/2019/05/remeber-web-guid-is-not-unique.html
---

It may seem weird, but yes - SharePoint Web GUID is not unique across the tenant/farm. And it has never been. <br />It is unique only inside a site collection.<br />It can be confusing (everyone knows what acronym means), most of us probably has never noticed that... But it is what it is.<br />And you'll probably notice it more often with fast site creation for modern Team and Comm sites in SharePoint Online.<br />So remember that fact... and modify your code if it based on assumption that the Guid is unique.<br />Few more words by Wictor Wilen: <a href="http://www.wictorwilen.se/when-a-guid-is-not-really-unique-i-m-looking-at-you-sharepoint" target="_blank">http://www.wictorwilen.se/when-a-guid-is-not-really-unique-i-m-looking-at-you-sharepoint</a><br /><br />That's all for today!<br />Have fun!