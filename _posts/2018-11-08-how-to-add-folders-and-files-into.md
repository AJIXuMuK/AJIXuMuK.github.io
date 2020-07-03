---
layout: post
title: How to Add Folders and Files into Master Page Gallery on Modern Team Sites
date: '2018-11-08T11:43:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Master Page Gallery
- PowerShell
- SharePoint Online Management Shell
- Access Denied
- Modern Team Sites
- DenyAddAndCustomizePages
- Permissions
- SharePoint
modified_time: '2018-11-13T13:07:06.231-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6582901642958654814
blogger_orig_url: http://blog.aterentiev.com/2018/11/how-to-add-folders-and-files-into.html
---

Sometimes, especially  when developing custom SharePoint Add-ins, we want to host assets (JavaScript, CSS, etc.) in Master Page Gallery.<br />This is a pretty standard approach that allows to reference the same resources from sub sites (I know that sub sites are now evil =)) but store everything in one place on root level of the site collection.<br /><b>Note: it is not necessary to host your assets in Master Page Gallery, it is one of the options. So, take it in mind, that you can use other options as well.</b>But what if we want to use that approach with Modern Team Sites?<br /><br /><a name='more'></a><br />First of all, you won't see <b>Master pages</b> menu item in Site Settings on Modern Team Site:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/web-designer-galleries.png" /><br />Well, okay, it's not a big deal as we know the url of the library: <span class="code">_catalogs/masterpage/Forms/AllItems.aspx</span><br />This url works and you are redirected to the standard Master Page Gallery: <img border="0" src="{{site.baseurl}}/assets/images/posts/2018/master-page-gallery.png" /><br />Now, if you try to create a folder here or upload a new file you'll get "Access Denied" error even if you are a Site Collection Admin:<br />Uploading a file:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/no-access.png" /></a><br />Adding a folder: <br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/folder-access-denied.png" /><br />So, let's go to Library Settings -&gt; Permissions for this document library (permissions are inherited) -&gt; Check Permissions.<br />The result of the permissions' check will look like that:<br /><img border="0" src="{{site.baseurl}}/assets/images/posts/2018/check-permissions.png" /><br />So, there is a "Deny" rule that restricts access to Master Page Gallery. And this rule is not a part of any Permission levels available on the site collection.<br />Actually, it's a site collection setting called <span class="code"><a href="https://msdn.microsoft.com/en-us/library/microsoft.online.sharepoint.tenantadministration.siteproperties.denyaddandcustomizepages.aspx" target="_blank">DenyAddAndCustomizePages</a></span>.<br />And we can change it using SharePoint Online Management Shell:<br />
<div markdown="1">
{% highlight console %}

Connect-SPOService https://&ltyourtenant>-admin.sharepoint.com
Set-SPOSite -Identity https://<yourtenant>.sharepoint.com/sites/<team-site> -DenyAddAndCustomizePages $false

{% endhighlight %}
</div>
After that the permissions in Master Page gallery work as expected and the Deny rule is gone.<br />Interesting thing that <span class="code">DenyAddAndCustomizePages</span> is equal to <span class="code">no-script</span> option - read the next <a href="https://docs.microsoft.com/en-us/sharepoint/allow-or-prevent-custom-script" target="_blank">article</a> for details.<br />And that's it for today!<br /><br />Have fun! 