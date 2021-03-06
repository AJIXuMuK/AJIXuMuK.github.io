---
layout: post
title: Reverting Back Custom Site Column Definition
date: '2018-01-18T09:42:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint 2013
- SharePoint 2016
- SchemaXml
- Site Column
- Feature
- Field Definition
- SharePoint
modified_time: '2018-01-18T09:42:24.419-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2720469383745360816
blogger_orig_url: http://blog.aterentiev.com/2018/01/reverting-back-custom-site-column.html
---

Sometimes it happens that the site column that was provision from a custom feature has been changed via code or using UI. In that case the column will receive a new version and will be untied for the feature. <br />And if later on you try to update the field from feature definition it won't work.<br />So let's look how to fix that situation. <br /><a name='more'></a><b>Very important thing to mention - the changes will be done directly in SharePoint Content Database. So <span style="color: red;">BACK IT UP</span> before proceeding with the steps.</b><br /><ol><li>Ok, the first thing to do is to get the Site Collection id. For example, using Powershell: 
<div markdown="1">
{% highlight console %}
$site = Get-SPSite("http://yoursite.com");
$site.Id

{% endhighlight %}
</div>
This cmdlets will display the Guid of the site into the management shell. <img border="0" src="{{site.baseurl}}/assets/images/posts/2018/guid.png" /></li><li>Next step is to open the Content Database related to the Site Collection in some IDE (most probably SQL Management Studio) and run the query listed below to check that the column was actually modified directly (not from feature): 
<div markdown="1">
{% highlight sql %}

SELECT *
  FROM [WSS_Content].[dbo].[ContentTypes]
  where Definition like '<Field%Name="FieldInternalName"%'
  and SiteId = 'site_guid'

{% endhighlight %}
</div>
where<br /><span class="code">WSS_Content</span> should be replaced with the name of Content Database<br /><span class="code">FieldInternalName</span> should be replaced with the field's internal name, and <br /><span class="code">site_guid</span> should be replaced with the site guid<br />If there is a modification in the field it will be listed by this query </li><li>Next step is to delete the Database entry: 
<div markdown="1">
{% highlight sql %}

DELETE
  FROM [WSS_Content].[dbo].[ContentTypes]
  where Definition like '<Field%Name="FieldInternalName"%'
  and SiteId = 'site_guid'

{% endhighlight %}
</div>
</li><li>And the last step is to republish/reactivate your solution/feature</li></ol>That's it! Now the field should be tied back to the feature.<br />In most scenarios these modifications may live in your environment forever. But sometimes they can lead to unpredicted issues.<br />For example, I recently got in situation when I needed to update the definition for Lookup field and is later used in subsites. The Lookup is provisioned in site scoped feature and is used in list definition that is provisioned in web scoped feature and activated in subsites.<br />As soon as I changed the XML for my Field Definition in the feature and deployed it, I got an exception that I can't activate a feature on a subsite.<br />The exception was pretty general: Value doesn't fall within expected range.<br />The reason for that was that SharePoint was comparing XML from the feature and from Site Column. And Site Column was modified outside the feature. So, XMLs were not equal.<br />Proceeding with the steps listed above I was able to fix the issue. <br /><br />Have fun!