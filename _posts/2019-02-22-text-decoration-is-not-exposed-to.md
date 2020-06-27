---
layout: post
title: text-decoration is not Exposed to Children
date: '2019-02-22T09:32:00.001-08:00'
author: Alex Terentiev
tags:
- markup
- CSS
- text-decoration
- html
- position
- fixed
- absolute
modified_time: '2019-02-22T09:32:42.030-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4515963297993850202
blogger_orig_url: http://blog.aterentiev.com/2019/02/text-decoration-is-not-exposed-to.html
---

Quick notice for myself and all others who will struggle with the issue that <span class="code">text-decoration</span> CSS property is not exposed to child elements.<br />It might be because of the <span class="code">position</span> value of the children. If it's set to <span class="code">absolute</span> or <span class="code">fixed</span> then parent value of <span class="code">text-decoration</span> will not take effect on such children. <br />That's it for today!<br />Have fun!