---
layout: post
title: Getting List's Incoming Email Settings From JavaScript
date: '2017-02-25T15:50:00.000-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint 2013
- SharePoint 2016
- SharePoint 2010
- CSOM
- SchemaXml
- SharePoint
modified_time: '2017-02-25T15:50:38.357-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-4993092547981052853
blogger_orig_url: http://blog.aterentiev.com/2017/02/getting-lists-incoming-email-settings.html
---

This task may be pretty unique but on one of the project I needed to get Incoming Email settings for all document libraries using JavaScript only.<br />After brief investigation I found that there are no any related properties in CSOM's/JSOM's List object.<br />But you can alway get list's Schema XML and get practically any information from there. Needed property is stored as attribute and has name EmailAlias.<br />So, the solution was pretty simple:<br />
<div markdown="1">
{% highlight javascript %}
// getting context
var ctx = SP.ClientContext.get_current();
// getting web
var web = ctx.get_web();
// getting list
var list = web.get_lists().getByTitle('Some Title');
// loading SchemaXml
ctx.load(list, 'SchemaXml');
ctx.executeQueryAsync(function() {
  // needed property is stored as an attribute with name EmailAlias
  // if the property is not empty then Incoming Email is configured for the list
  var content = (new window.DOMParser()).parseFromString(list.get_schemaXml(), 'text/xml');
  var emailAlias = content.getElementsByTagName('List')[0].getAttribute('EmailAlias');
});

{% endhighlight %}
</div>
<br />Using this approach you can also get other properties as well. Some of them are also included in JSOM's object as properties but some may not.<br /><br />Have fun!