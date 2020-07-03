---
layout: post
title: '"Open with Explorer" and "Upload Multiple Documents" links availability check'
date: '2014-11-25T10:40:00.001-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint UI
- SharePoint 2013
- ASP.NET
- SharePoint 2010
- Open with Explorer
- Upload multiple documents
- SharePoint
modified_time: '2014-11-29T19:54:59.138-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2798574779823769056
blogger_orig_url: http://blog.aterentiev.com/2014/11/open-with-explorer-availability.html
---

<div dir="ltr" style="text-align: left;" trbidi="on"><div dir="ltr" style="text-align: left;" trbidi="on"><div dir="ltr" style="text-align: left;" trbidi="on">When you create custom UI for SharePoint libraries you probably may need to create your own "Open with Explorer" and "Upload Multiple Documents" (the second link and functionality is available only in SharePoint 2010, not 2013) links. There are rules when these links are visible and operations are enabled: it should be IE and even not any version of it. You can search the rules and write your own code or use javascript function that is written by Microsoft and used in out-of-the-box link. It's named <span class="code">SupportsNavigateHttpFolder</span><br /><span style="font-family: Times, Times New Roman, serif;">You can use it something like that on server side:</span><br /><div class="p1"><br /></div></div></div>
<div markdown="1">
{% highlight csharp %}
ScriptManager.RegisterStartupScript(this, this.GetType(), "CheckOpenInExplorerAvailability",  
@"<script type='text/javascript'>  
var displayOpenInExp = SupportsNavigateHttpFolder() ? 'block' : 'none';   
var elOpenInExp = document.getElementById('" + controlId + @"');   
elOpenInExp.style.display = displayOpenInExp;  
</script>", false);
{% endhighlight %}
</div>
or on the client (here I'm using jquery):<br />
<div markdown="1">
{% highlight javascript %}
$('#' + controlId)[(SupportsNavigateHttpFolder() ? 'show' : 'hide']();
{% endhighlight %}
</div>
Have fun!</div>