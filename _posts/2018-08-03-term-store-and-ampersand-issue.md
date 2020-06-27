---
layout: post
title: Term Store and Ampersand (&) Issue
date: '2018-08-03T12:54:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SharePoint 2013
- SharePoint 2016
- Taxonomy
- Managed Metadata
- Bug
- Issue
- O365
- Office 365
- Term Store
- SharePoint
modified_time: '2018-08-03T12:54:14.056-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-9180452703527883033
blogger_orig_url: http://blog.aterentiev.com/2018/08/term-store-and-ampersand-issue.html
---

When working with Term Store take into consideration that SharePoint for some reason replaces ampersand character&nbsp;&amp; (code 38) with ampersand character&nbsp;＆ (code 65286) in the label. The bad thing is that if you create a Term with the name with &amp; you won't be able to change it back from symbol 65286 to symbol 38 as default label of the term can't be modified.<br />Simple way to play with it:<br /><br /><ol><li>Create new term "Wood &amp; Fire"</li><li>Copy the default label text</li><li>Paste the value as string into browser console and run call charCodeAt(5):&nbsp;"Wood ＆ Fire".charCodeAt(5)</li><li>The result will be 65286 instead of 38</li></ol><div>I tested the behavior on O365, SP 2016 and SP2013 - all the same.</div><div><br /></div><div>Have fun!</div>