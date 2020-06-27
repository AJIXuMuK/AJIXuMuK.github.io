---
layout: post
title: System.Uri and Site.OpenWeb
date: '2019-01-25T11:40:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint 2019
- SharePoint Online
- SharePoint 2013
- SharePoint 2016
- SharePoint 2010
- C#
- Client Side Object Model
- CSOM
- SharePoint
modified_time: '2019-01-25T11:45:32.480-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6077955969848952286
blogger_orig_url: http://blog.aterentiev.com/2019/01/systemuri-and-siteopenweb.html
---

Often we're in a situation when we need to get a server relative url from absolute url.<br />One of the easiest ways to do that in C# is to use <span class="code">System.Uri</span> object like that: 
<div markdown="1">
{% highlight csharp %}

public static string GetRelativeUrl(string absoluteUrl)
{
  return new Uri(absoluteUrl).AbsolutePath;
}

{% endhighlight %}
</div>
One thing we should remember about this method is: <span class="code">System.Url</span> automatically escapes the url. As a result, as example, <span class="code">https://tenant.sharepoint.com/sites/mysite/Site With Spaces</span> will be replaced with <span class="code">https://tenant.sharepoint.com/sites/mysite/Site%20With%20Spaces</span>, and <span class="code">AbsolutePath</span> will return <span class="code">/sites/mysite/Site%20With%20Spaces</span>.<br />And that's critical for SharePoint. If you provide escaped server relative url to <span class="code">Site.OpenWeb</span> (or <span class="code">SPSite.OpenWeb</span> for server OM) you'll get <b>File not found</b> exception for CSOM and something like <b>There is no Web named</b> for Server Object Model.<br />To avoid that and still use <span class="code">System.Uri</span> I would recommend to use <span class="code">Uri.UnescapeDataString</span> static method: 
<div markdown="1">
{% highlight csharp %}

public static string GetRelativeUrl(string absoluteUrl)
{
  return Uri.UnescapeDataString(new Uri(absoluteUrl).AbsolutePath);
}

{% endhighlight %}
</div>
It will revert back the escaping.<br />Hope that would help!<br />That's it for today!<br />Have fun! 