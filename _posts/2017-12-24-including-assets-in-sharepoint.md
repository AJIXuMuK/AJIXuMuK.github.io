---
layout: post
title: Including Assets in SharePoint Framework Solution Packages
date: '2017-12-24T12:26:00.001-08:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Client Side Web Part
- O365
- SharePoint Framework
- Office 365
- includeClientSideAssets
- SharePoint
modified_time: '2017-12-27T10:45:42.661-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6709808457447562490
blogger_orig_url: http://blog.aterentiev.com/2017/12/including-assets-in-sharepoint.html
---

<a href="https://dev.office.com/blogs/sharepoint-framework-1-4-and-new-site-updates-now-available">SharePoint Framework v1.4</a> contains new feature - ability to include all the used assets inside solution package to simplify the overall process of the solution hosting and delivery.<br />The assets are all JavaScript files, CSS files, images, fonts, etc. that are referenced in the project.<br />Let's deep dive a bit in the topic to understand the process. <br /><a name='more'></a><h2>Configuration</h2>First of all, there is a new boolean property in <span class="code">package-solution.json</span> file named <span class="code">includeClientSideAssets</span>. By default this setting is set to true which means that all the assets <b>will be included</b> in the <span class="code">.sppkg</span> file.<br />There is one more condition to include the assets in the package: <span class="code">cdnBasePath</span> property in <span class="code">write-manifests.json</span> file should remain the default one: <span class="code">&lt;!-- PATH TO CDN --&gt;</span>. If we change it to some custom url, we'll see next warning while running <span class="code">gulp package-solution --ship</span>: <br />
<div markdown="1">
{% highlight console %}
Warning - [package-solution] The "cdnBasePath" in "config/write-manifests.json" has been changed from its default value ("<!-- PATH TO CDN -->") to "https://cdn.test.com", however the "includeClientSideAssets" setting in "config/package-solution.json" is "true" and will be ignored. If you meant to deploy your assets in your SPPKG to SharePoint, reset the value of "cdnBasePath" to "<!-- PATH TO CDN -->".

{% endhighlight %}
</div>
If both settings are set correctly, all the assets will be included in the package. <br /><h2>Deployment and Hosting</h2>No we have the <span class="code">.sppkg</span> file with all the assets inside. This package can be deployed either to Tenant App Catalog or to <a href="https://support.office.com/en-us/article/Manage-the-Site-Collection-App-Catalog-928b9b61-a9de-4563-a7d1-6231aa9d4d19">Site Collection App Catalog</a>.<br />After deployment the assets will be hosted from one of next locations (based on the tenant settings): <br /><ul><li>Office 365 Public CDN - if the CDN is configured (See details about O365 Public CDN <a href="https://dev.office.com/blogs/general-availability-of-office-365-cdn">here</a>). </li><li>Directly from App Catalog (Tenant App Catalog or Site Collection App Catalog). The url will look like <span class="code">https://orgname.sharepoint.com/sites/&lt;app_catalog_url_or_site_collection_url&gt;/CliendSideAssets/&lt;solution-id&gt;</span></li></ul><h2>Benefits</h2>I see next main benefits of including assets to the solution: <br /><ul><li>Organizations (tenants owners) are in control of the assets - you know what is running on the tenant</li><li>No need of additional hosting environment</li><li>The same package can be deployed to multiple environments (QA, UAT, PROD or multiple customers)</li><li>As ISV or product owner you can create unique package for every customer with some unique configurations</li></ul>I would say that in most scenarios this option is the one to use. And only in some cases you can use external CDN. <br />That's it!<br />Have fun!