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
thumbnail: https://3.bp.blogspot.com/-IJG439-XCEo/W-SLChDfKdI/AAAAAAAAA9E/lq3LXzNHTu09x5cNdO21DjVqu6gXO9B6QCLcBGAs/s72-c/Screen%2BShot%2B2018-11-08%2Bat%2B11.12.44%2BAM.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6582901642958654814
blogger_orig_url: http://blog.aterentiev.com/2018/11/how-to-add-folders-and-files-into.html
---

Sometimes, especially  when developing custom SharePoint Add-ins, we want to host assets (JavaScript, CSS, etc.) in Master Page Gallery.<br />This is a pretty standard approach that allows to reference the same resources from sub sites (I know that sub sites are now evil =)) but store everything in one place on root level of the site collection.<br /><b>Note: it is not necessary to host your assets in Master Page Gallery, it is one of the options. So, take it in mind, that you can use other options as well.</b>But what if we want to use that approach with Modern Team Sites?<br /><br /><a name='more'></a><br />First of all, you won't see <b>Master pages</b> menu item in Site Settings on Modern Team Site:<br /><a href="https://3.bp.blogspot.com/-IJG439-XCEo/W-SLChDfKdI/AAAAAAAAA9E/lq3LXzNHTu09x5cNdO21DjVqu6gXO9B6QCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.12.44%2BAM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-IJG439-XCEo/W-SLChDfKdI/AAAAAAAAA9E/lq3LXzNHTu09x5cNdO21DjVqu6gXO9B6QCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.12.44%2BAM.png" data-original-width="1600" data-original-height="1180" width="600" /></a><br />Well, okay, it's not a big deal as we know the url of the library: <span class="code">_catalogs/masterpage/Forms/AllItems.aspx</span><br />This url works and you are redirected to the standard Master Page Gallery: <a href="https://4.bp.blogspot.com/-Hiw3tzQNPZw/W-SLj2yF2gI/AAAAAAAAA9M/nvBvKqPfO4sSIg0r3w_K-r_4r2Nq9p8MQCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.15.40%2BAM.png" imageanchor="1" ><img border="0" src="https://4.bp.blogspot.com/-Hiw3tzQNPZw/W-SLj2yF2gI/AAAAAAAAA9M/nvBvKqPfO4sSIg0r3w_K-r_4r2Nq9p8MQCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.15.40%2BAM.png" data-original-width="1600" data-original-height="426" width="600" /></a><br />Now, if you try to create a folder here or upload a new file you'll get "Access Denied" error even if you are a Site Collection Admin:<br />Uploading a file:<br /><a href="https://3.bp.blogspot.com/-3jNrsQ-JnMM/W-SOXHxv2wI/AAAAAAAAA9k/vU778Jk7_pUsrl0plQt5kUkAE1v2SLr7gCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.28.00%2BAM.png" imageanchor="1" ><img border="0" src="https://3.bp.blogspot.com/-3jNrsQ-JnMM/W-SOXHxv2wI/AAAAAAAAA9k/vU778Jk7_pUsrl0plQt5kUkAE1v2SLr7gCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.28.00%2BAM.png" data-original-width="1600" data-original-height="622" width="600" /></a><br />Adding a folder: <br /><a href="https://2.bp.blogspot.com/-LFAJPWWGl3M/W-SObR20QAI/AAAAAAAAA9o/FE2tsTJurP8iaotCyepjD023NYviiYdGwCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.28.18%2BAM.png" imageanchor="1" ><img border="0" src="https://2.bp.blogspot.com/-LFAJPWWGl3M/W-SObR20QAI/AAAAAAAAA9o/FE2tsTJurP8iaotCyepjD023NYviiYdGwCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.28.18%2BAM.png" data-original-width="1452" data-original-height="832" width="600" /></a><br />So, let's go to Library Settings -&gt; Permissions for this document library (permissions are inherited) -&gt; Check Permissions.<br />The result of the permissions' check will look like that:<br /><a href="https://2.bp.blogspot.com/-b8_uPLyeRyY/W-SPcf8LXqI/AAAAAAAAA94/THp8o3h8zMETJ8h-A_eZFQa0Yt6UhPSeQCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.32.25%2BAM.png" imageanchor="1" ><img border="0" src="https://2.bp.blogspot.com/-b8_uPLyeRyY/W-SPcf8LXqI/AAAAAAAAA94/THp8o3h8zMETJ8h-A_eZFQa0Yt6UhPSeQCLcBGAs/s1600/Screen%2BShot%2B2018-11-08%2Bat%2B11.32.25%2BAM.png" data-original-width="1414" data-original-height="796" width="600" /></a><br />So, there is a "Deny" rule that restricts access to Master Page Gallery. And this rule is not a part of any Permission levels available on the site collection.<br />Actually, it's a site collection setting called <span class="code"><a href="https://msdn.microsoft.com/en-us/library/microsoft.online.sharepoint.tenantadministration.siteproperties.denyaddandcustomizepages.aspx" target="_blank">DenyAddAndCustomizePages</a></span>.<br />And we can change it using SharePoint Online Management Shell:<br />
<div markdown="1">
{% highlight console %}

Connect-SPOService https://&ltyourtenant>-admin.sharepoint.com
Set-SPOSite -Identity https://<yourtenant>.sharepoint.com/sites/<team-site> -DenyAddAndCustomizePages $false

{% endhighlight %}
</div>
After that the permissions in Master Page gallery work as expected and the Deny rule is gone.<br />Interesting thing that <span class="code">DenyAddAndCustomizePages</span> is equal to <span class="code">no-script</span> option - read the next <a href="https://docs.microsoft.com/en-us/sharepoint/allow-or-prevent-custom-script" target="_blank">article</a> for details.<br />And that's it for today!<br /><br />Have fun! 