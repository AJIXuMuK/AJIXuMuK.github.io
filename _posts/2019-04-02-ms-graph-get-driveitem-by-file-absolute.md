---
layout: post
title: 'MS Graph: Get DriveItem by File Absolute Url'
date: '2019-04-02T12:15:00.000-07:00'
author: Alex Terentiev
tags:
- DriveItem
- Microsoft Graph
- SharePoint Online
- MS Graph
- Office Development
- O365
- Office 365
- OneDrive
- Absolute Url
- SharePoint
modified_time: '2019-04-02T12:15:58.884-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-5430537705948008571
blogger_orig_url: http://blog.aterentiev.com/2019/04/ms-graph-get-driveitem-by-file-absolute.html
---

 Microsoft Graph allows to work with OneDrive files and SharePoint documents using <span class="code">DriveItem</span> resource type. Using this entity you can read information about a file, such as size, content, file type, and so on (here is a <a href="https://docs.microsoft.com/en-us/graph/api/resources/driveitem?view=graph-rest-1.0" target="_blank">link</a> to the resource type documentation).<br />You can get <span class="code">DriveItem</span> object from a specific <span class="code">Drive</span>, <span class="code">Site</span>, <span class="code">Group</span>, or user's connected OneDrive. But for all of these operations you need to know identifiers of parent resources, e.g. drive id, or site id (<a href="https://docs.microsoft.com/en-us/graph/api/driveitem-get?view=graph-rest-1.0" target="_blank">documentation</a> on how to get drive item).<br />But what if the file is located in a custom document library, in a subfolder, and on a subsite? And you have only absolute url for the document, like <span class="code">https://your-tenant.sharepoint.com/sites/your-site/your-subsite/DocLibrary/subfolder/file.ext</span>? <br /><a name='more'></a><br />If you try to use the approach described in the documentation, you'll need to: <ol><li>Correctly split the url to have site relative url, document library url, and file path in that library. And the algorithm for that might be really complicated, especially without additional knowledge about site contents.</li><li>Get all drives from the site and find <span class="code">drive</span> and its id associated with the document library (unfortunately there is no API to get <span class="code">drive</span> by path like <span class="code">/sites/{site-id}/drives:/path-to-doc-library</span>)</li><li>Get <span class="code">driveItem</span> using request like <span class="code">/sites/{your-tenant}:/{site-relative-url}/drives/{drive-id}/root:/{item-path}</span></li></ol>All of these look really complicated both from implementation standpoint and number of requests to be done...<br />But... Good news everyone! And the name of it - MS Graph <span class="code">shares</span> endpoint!<br />We can actually transform our absolute url into sharing encoded url and use it with <span class="code">/shares</span> endpoint to get <span class="code">driveItem</span>.<br />So, first, let's encode the url, using the code from <a href="https://docs.microsoft.com/en-us/graph/api/shares-get?view=graph-rest-1.0#encoding-sharing-urls" target="_blank">official documentation</a>: 
<div markdown="1">
{% highlight csharp %}

string base64Value = System.Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(fileAbsoluteUrl));
string encodedUrl = "u!" + base64Value.TrimEnd('=').Replace('/','_').Replace('+','-');

{% endhighlight %}
</div>
<b>Note:</b> for JavaScript/TypeScript documentation you can use <a href="https://www.w3schools.com/jsref/met_win_btoa.asp" target="_blank">btoa</a> function.<br />Now, when we have the sharing url we can get the <span class="code">driveItem</span>: 
<div markdown="1">
{% highlight console %}

GET https://graph.microsoft.com/v1.0/shares/{sharing-url}/driveItem

{% endhighlight %}
</div>
Moreover, if we're working with SharePoint document we can also get related ListItem and all the fields: 
<div markdown="1">
{% highlight console %}

GET https://graph.microsoft.com/v1.0/shares/{sharing-url}/driveItem?$expand=listItem($expand=fields)

{% endhighlight %}
</div>
So, using single request and simple code we can get all the information we need! That's just awesome! <h2>Conclusion</h2>The conclusion here is one sentence: if you need to get drive item by absolute url, use <span class="code">shares</span> endpoint! <br /><br />That's all for today!<br />Have fun! 