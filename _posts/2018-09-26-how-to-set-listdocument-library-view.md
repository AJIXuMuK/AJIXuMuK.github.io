---
layout: post
title: 'How to: Set List/Document Library View Page as Modern Team Site Welcome  Page
  (Homepage)'
date: '2018-09-26T20:42:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- JSOM
- C#
- Welcome Page
- Office 365
- Homepage
- PnP
- PnP PowerShell
- PowerShell
- O365
- Client Side Object Model
- CSOM
modified_time: '2019-07-23T08:57:34.387-07:00'
thumbnail: https://3.bp.blogspot.com/-niIlteglGtg/W6xLBdlptdI/AAAAAAAAA7M/fZJYUTbrmoQw25rOLj1unrt-GDchGjeLQCLcBGAs/s72-c/Homepage.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6229683554851032521
blogger_orig_url: http://blog.aterentiev.com/2018/09/how-to-set-listdocument-library-view.html
---

When you need to change the Home Page of Modern Team Site in SharePoint Online it's pretty easy to do through UI.<br />Just go to the Pages library, open context menu for the page you want to make a homepage and select "Make homepage"<br /><a href="https://3.bp.blogspot.com/-niIlteglGtg/W6xLBdlptdI/AAAAAAAAA7M/fZJYUTbrmoQw25rOLj1unrt-GDchGjeLQCLcBGAs/s1600/Homepage.png" imageanchor="1"><img border="0" data-original-height="934" data-original-width="1177" src="https://3.bp.blogspot.com/-niIlteglGtg/W6xLBdlptdI/AAAAAAAAA7M/fZJYUTbrmoQw25rOLj1unrt-GDchGjeLQCLcBGAs/s1600/Homepage.png" width="600" /></a><br />But what if the page you want to make a homepage is a Document Library or List View Page that is, obviously, not presented in the Pages library?<br /><a name='more'></a>One of the ideas that could get into a mind is to use "Welcome page" functionality from Classic Publishing SharePoint site.<br />Even if the Publishing feature is not activated, the page is available and accessible using url (relative to the site) <span class="code">_layouts/15/AreaWelcomePage.aspx</span>.<br />But if you try to set the url to library's View page there you'll get the error listed below:<br /><a href="https://1.bp.blogspot.com/-jjOOCfCkZSM/W6xNKcr25rI/AAAAAAAAA7k/Ol38DKDUPVIIP5pGrv6BNOyO3zZcD96DQCLcBGAs/s1600/homepage-error.png" imageanchor="1" ><img border="0" src="https://1.bp.blogspot.com/-jjOOCfCkZSM/W6xNKcr25rI/AAAAAAAAA7k/Ol38DKDUPVIIP5pGrv6BNOyO3zZcD96DQCLcBGAs/s1600/homepage-error.png" data-original-width="1466" data-original-height="205" width="600" /></a><br />Thankfully, we have <a href="https://github.com/SharePoint/PnP-PowerShell" target="_blank">PnP PowerShell</a>, CSOM and JSOM:<br />
<div markdown="1">
{% highlight console %}

Connect-PnPOnline https://yourcompany.sharepoint.com/sites/yoursitecollection
$site = Get-PnPSite
$web = $site.OpenSite("WebRelativeUrl") # you can use $site.RootSite for root site of the site collection
$folder = $web.RootFolder
$folder.WelcomePage = "Shared%20Documents/Forms/AllItems.aspx"
$folder.Update()
$web.Update()
Invoke-PnPQuery

{% endhighlight %}
</div>
That's all you need. After executing the script you'll have you homepage set to Documents All Items View page.<br />Same using CSOM: 
<div markdown="1">
{% highlight csharp %}

using(var ctx = new ClientContext("https://yourcompany.sharepoint.com/sites/yoursite"))
{
  ctx.Credentials = new SharePointOnlineCredentials(username, password);
  var site = ctx.Site;
  var web = site.OpenWeb("WebRelativeUrl"); // you can use site.RootWeb for root site of the site collection
  var rootFolder = web.RootFolder;
  rootFolder.WelcomePage = "Shared%20Documents/Forms/AllItems.aspx";
  rootFolder.Update();
  web.Update();
  ctx.ExecuteQuery();
}

{% endhighlight %}
</div>
Or, using JSOM: 
<div markdown="1">
{% highlight javascript %}

var ctx = SP.ClientContext.get_current();
var site = ctx.get_site();
var web = site.openWeb('WebRelativeUrl'); // you can use site.get_rootWeb() for root site of the site collection
var folder = web.get_rootFolder();
folder.set_welcomePage('Shared%20Documents/Forms/AllItems.aspx');
folder.update();
web.update();
ctx.executeQueryAsync(() => { console.log('success'); }, () => { console.log('error'); });

{% endhighlight %}
</div>
<br />That's it!<br /><br />Have fun! 