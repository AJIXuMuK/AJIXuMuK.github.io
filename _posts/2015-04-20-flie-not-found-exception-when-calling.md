---
layout: post
title: Flie Not Found exception when calling ExecuteQuery in Client Side Object Model.
date: '2015-04-20T18:18:00.001-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SharePoint 2013
- ClientContext.ExecuteQuery
- Client Side Object Model
- CSOM
- SharePoint Apps
- File Not Found
- SharePoint
modified_time: '2015-04-20T18:18:57.367-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4871611910882696300
blogger_orig_url: http://blog.aterentiev.com/2015/04/flie-not-found-exception-when-calling.html
---

Today I've got an exception in my Provider-Hosted App. The message was "File not found" and nothing more...<br />But I do not work with files, document libraries, attachments or any other resources that can be received as a file in that app. That why such a message was very unexpected.<br />After debugging I understand where was a problem: I was trying to open Web with absolute Url, not relative.<br />So, if you have the same exception check your Urls.<br />Have fun!