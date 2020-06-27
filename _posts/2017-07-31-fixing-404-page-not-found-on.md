---
layout: post
title: Fixing "404 Page not found" on New/Edit/Display Forms and List Views After
  Custom Web Template Site Migration (2010 -> 2013)
date: '2017-07-31T17:10:00.003-07:00'
author: Alex Terentiev
tags:
- SharePoint 2013
- Web Template
- Site Definition
- SharePoint 2010
- C#
- '404'
- Migration
- WebTemplate
- onet.xml
- Site Collection Upgrade
- SharePoint
modified_time: '2017-09-08T11:03:35.097-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4913103193047792588
blogger_orig_url: http://blog.aterentiev.com/2017/07/fixing-404-page-not-found-on.html
---

SharePoint migration is always a challenge. And the more customizations you have the more challenging it is.<br />In this post I want to describe the solution to fix "404 Page not found" issue that may occur after migrating sites based on custom web templates and custom features.<br /><br /><a name='more'></a>Let's say you have a product that works both in SharePoint 2010 and 2013.<br /><div>It contains custom Site Definition, custom Web Templates and a bunch of custom list definitions and instances.</div><div>Some of your customers want to migrate from 2010 to 2013.</div><div>Everything looks pretty straightforward:</div><div><ul><li>backup content database on 2010</li><li>restore it on 2013</li><li>install the product's solution with CompatibilityMode 14, 15</li><li>mount the database</li><li>proceed Site Collection Upgrade (either from UI or with PowerShell script)</li><li>happily use migrated content in SharePoint 2013.</li></ul><div>But after doing all these steps I've received "404 Page not found" error for each and every page that was related to custom lists defined in the product's package.</div></div><div>Logs contained next message:</div><div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Relying on fallback logic in VghostPageManager::getGhostDocument() for document: 'C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\Template\Features\CustomFeatureName\CustomListName\AllItems.aspx'</span></div><div>The weird thing here was to see 14 Hive folder instead of 15. Potentially it could mean that the list or site was not upgraded for some reason.</div><div>Thankfully, there is a separate log file for Site Collection Upgrade process and it contained a lot of warnings like:</div><div><div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: small;">Template CustomWebTemplateName#0: Web template upgrade for web/site &lt;url&gt; returned NeedsUpgrade false. Not upgrading. Web template:&nbsp;CustomWebTemplateName#0, web template version: 4.0.0.2, target web template version: 15.0.0.2</span></div></div><div>This message is logged in <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">SPWebTempateSequence.DoUpgrade()</span> if <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">SPWebTemplateSequence.NeedsUpgrade</span> property returns <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">false</span>.</div>
<div markdown="1">
{% highlight csharp %}
protected override void DoUpgrade()
{
  if (!this.NeedsUpgrade)
  {
    base.Log.InfoTag(0x258845, string.Format(CultureInfo.InvariantCulture, this.LogPreamble + "Web template upgrade for web/site {0} returned NeedsUpgrade false. Not upgrading. Web template: {1}, web template version: {2}, target web template version: {3}", new object[] { this.Web.Url, this.WebTemplate.Name, this.WebTemplateVersion, this.TargetWebTemplateVersion }));
  }
  // rest of the method
}

{% endhighlight %}
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">NeedsUpgrade</span> getter looks like that:<br />
<div markdown="1">
{% highlight csharp %}
public override bool NeedsUpgrade
{
  get
  {
    bool flag = false;
    if (((this.WebTemplate != null) &amp;&amp; (this.WebTemplateVersion < this.TargetWebTemplateVersion)) &amp;&amp; (this.XmlConfiguration != null))
    {
      flag = true;
    }
    // Logging goes here
    
    return flag;
  }
}

{% endhighlight %}
</div>
</div><div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplate</span> is not null if the web template's xml is found in&nbsp;c:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\1033\XML\ (1033 here is an En-US locale identifier)<br /><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplateVersion</span> contains '4' as major revision for SharePoint 2010-based content, <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">TargetWebTemplateVersion</span> contains '15' as major revision as we're upgrading to SharePoint 2013.<br />So, first 2 conditions are true.<br />The last condition is the most interesting and complicated.<br /><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">XmlConfiguration</span> getter implementation looks like that:<br />
<div markdown="1">
{% highlight csharp %}
internal SPXmlWebTemplateConfiguration XmlConfiguration
{
  get
  {
    if (this.m_xwtcWebTemplate == null)
    {
      foreach (SPXmlConfiguration configuration in SPXmlConfigurationManager.GetInstanceByCompatibilityLevel(this.TargetMajorVersion).SelectXmlConfigurations(SPXmlConfiguration.WebTemplateUpgradeXPath))
      {
        List<spxmlwebtemplateconfiguration> webTemplateConfigurations = configuration.GetWebTemplateConfigurations(this.WebTemplateID, this.LCID);
        int revision = this.TargetWebTemplateVersion.Revision;
        foreach (SPXmlWebTemplateConfiguration configuration2 in webTemplateConfigurations)
        {
          if (configuration2.IsApplicable(this))
          {
            this.m_xwtcWebTemplate = configuration2;
            break;
          }
        }
      }
    }
    return this.m_xwtcWebTemplate;
  }
}

{% endhighlight %}
</div>
</div><div>Further investigation of that code leads to next conclusions:<br /><ul><li><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">SPXmlConfigurationManager</span> loads xml configuration files from&nbsp;c:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\CONFIG\UPGRADE\ folder</li><li>Each file can contain multiple (or none) <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplate</span> elements (Upgrade Definition schema is described&nbsp;<a href="https://msdn.microsoft.com/EN-US/library/office/ms412955.aspx">here</a>)</li><li>Each found <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplate</span> is tested if it is applicable to the Web Template that is being processed by Site Upgrade action.</li><li>If it is applicable then the Web Template (and the site) will be upgraded.</li></ul><div>Then all we need is to add an XML file with a <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplate</span> element to c:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\CONFIG\UPGRADE\. Also, we need to look at the code of <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">IsApplicable</span> method to understand what values to provide in our <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">WebTemplate</span> element:</div>
<div markdown="1">
{% highlight csharp %}
flag = ((this.FromProductVersion == webTemplateVersion.Major)
  &amp;&amp; (this.ToSchemaVersion == targetWebTemplateVersion.Revision)) 
  &amp;&amp; ((this.BeginFromSchemaVersion <= webTemplateVersion.Revision) 
  &amp;&amp; (webTemplateVersion.Revision <= this.EndFromSchemaVersion));

{% endhighlight %}
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">FromProductVersion</span> should be equal to '4' as we're upgrading from SharePoint 2010.<br /><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ToSchemaVersion</span> should be equal to the <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Revision</span> number from the site definition <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onet.xml</span> file applied to SharePoint 2013.<br /><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">BeginFromSchemaVersion</span> and <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">EndFromSchemaVersion</span> should form a range that contain the <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Revision</span> from the site definition <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">onet.xml</span> file applied to SharePoint 2010. I would suggest to set <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">BeginFromSchemaVersion</span> to 0 and <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">EndFromSchemaVersion</span> to some large number, for example, 10.<br />In my case resulting XML looked like that:</div>
<div markdown="1">
{% highlight xml %}
<Config xmlns="urn:Microsoft.SharePoint.Upgrade">
<WebTemplate
  ID="<template_id>"
  LocaleId="*"
  FromProductVersion="4"
  BeginFromSchemaVersion="0"
  EndFromSchemaVersion="10"
  ToSchemaVersion="2">
</WebTemplate>
</Config>
</template_id>
{% endhighlight %}
</div>
This file must be placed to c:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\CONFIG\UPGRADE\ <b>on each Web Front-End server before the Site Collection Upgrade has been started</b>.<br />Either you can just copy the file to needed location as it is one-time operation, or you can include mapped folder to your Visual Studio project and deploy the file during solution installation.<br />This solution works right at the moment of the migration and Site Collection Upgrade. But if some customer has already migrated the content and upgraded the site collection then the only way to fix the issue is to execute SQL script to update <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">SetupPathVersion</span> value for all the items in the site collection:</div>
<div markdown="1">
{% highlight sql %}
UPDATE [dbo.AllDocs]
SET SetupPathVersion = '15'
WHERE SiteId = '<site_collection_id>' AND 'SetupPathVersion' = '4'

{% endhighlight %}
</div>
<br /><b>UPDATE</b><ul><li> If you update from 2007 to 2010 and then to 2013 SetupPathVersion will contain '3' instead of '4'. It means that above script should be modified to update entries that have '3' as SetupPathVersion: 
<div markdown="1">
{% highlight sql %}
UPDATE [dbo.AllDocs]
SET SetupPathVersion = '15'
WHERE SiteId = '<site_collection_id>' AND ('SetupPathVersion' = '4' OR 'SetupPathVersion' = '3')

{% endhighlight %}
</div>
</li><li>If your Feature for some reason had different name in previous versions you should also update 'SetupPath' with script like: 
<div markdown="1">
{% highlight sql %}
UPDATE [dbo.AllDocs]
SET SetupPath = CAST(REPLACE(CAST(SetupPath as nvarchar(MAX)), 'old_feature_name', 'new_feature_name') as nvarchar(255))
WHERE SiteId = '<site_collection_id>' AND 'SetupPath' LIKE 'Features\old_feature_nane%'

{% endhighlight %}
</div>
</li></ul>Hope this will save you some time.<br />Please, feel free to leave a comment if you have any questions.<br />Have fun! 