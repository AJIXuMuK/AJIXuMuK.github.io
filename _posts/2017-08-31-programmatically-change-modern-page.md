---
layout: post
title: Programmatically Change Modern Page Banner Image
date: '2017-08-31T18:00:00.001-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- JSOM
- Modern Page
- O365
- CSOM
- SharePoint Framework
- Office 365
- Future of SharePoint
- SharePoint
modified_time: '2017-08-31T18:00:56.469-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-8683812211852752162
blogger_orig_url: http://blog.aterentiev.com/2017/08/programmatically-change-modern-page.html
---

In this post I want to describe how to programmatically change Modern Page Banner Image<br />The code below is written on JavaScript using JSOM (JavaScript Client Object Model) but it can be easily converted to C# (CSOM) and PowerShell as well.<br /><a name='more'></a>Let's say you've created some site using any provisioning mechanism (for example, PnP Provisioning Engine). And you also created multiple modern pages on the site and want to change the banner image for all of them.<br />After some investigation I found a way to achieve that: list items in Site Pages library have a field named <span class="code">LayoutWebpartsContent</span>. If you look at the value of this field you'll see something like: <br />
<div markdown="1">
{% highlight html %}
<div>
  <div data-sp-controldata="%7B%22id%22&amp;#58;%22cbe7b0a9-3504-44dd-a3a3-0e5cacd07788%22,%22instanceId%22&amp;#58;%22cbe7b0a9-3504-44dd-a3a3-0e5cacd07788%22,%22title%22&amp;#58;%22Title%20Region%22,%22description%22&amp;#58;%22Title%20Region%20Description%22,%22serverProcessedContent%22&amp;#58;%7B%22htmlStrings%22&amp;#58;%7B%7D,%22searchablePlainTexts%22&amp;#58;%7B%7D,%22imageSources%22&amp;#58;%7B7D,%22links%22&amp;#58;%7B%7D%7D,%22dataVersion%22&amp;#58;%221.0%22,%22properties%22&amp;#58;%7B%22title%22&amp;#58;%22Logo%20test%22,%22imageSourceType%22&amp;#58;4,%22translateX%22&amp;#58;50,%22translateY%22&amp;#58;50%7D%7D" data-sp-canvascontrol=""></div>
</div>

{% endhighlight %}
</div>
So it's an HTML markup that contains interesting attribute <span class="code">data-sp-controldata</span>. It looks like some escaped JSON.<br />Let's unescape it and also replace <span class="code">&amp;#58;</span> with <span class="code">:</span>. The result looks like that: <br />
<div markdown="1">
{% highlight javascript %}
{
    "id": "cbe7b0a9-3504-44dd-a3a3-0e5cacd07788",
    "instanceId": "cbe7b0a9-3504-44dd-a3a3-0e5cacd07788",
    "title": "Title Region",
    "description": "Title Region Description",
    "serverProcessedContent": {
        "htmlStrings": {},
        "searchablePlainTexts": {},
        "imageSources": {},
        "links": {}
    },
    "dataVersion": "1.0",
    "properties": {
        "title": "Logo test",
        "imageSourceType": 4,
        "translateX": 50,
        "translateY": 50
    }
}

{% endhighlight %}
</div>
If you proceed the same steps for the page that contains some banner image, you'll see a bit different result: <br />
<div markdown="1">
{% highlight javascript %}
{
    "id": "cbe7b0a9-3504-44dd-a3a3-0e5cacd07788",
    "instanceId": "cbe7b0a9-3504-44dd-a3a3-0e5cacd07788",
    "title": "Title Region",
    "description": "Title Region Description",
    "serverProcessedContent": {
        "htmlStrings": {},
        "searchablePlainTexts": {},
        "imageSources": {
            "imageSource": "/sites/contoso/SiteAssets/SitePages/banner.png"
        },
        "links": {}
    },
    "dataVersion": "1.0",
    "properties": {
        "title": "Logo test",
        "imageSourceType": 2,
        "translateX": 50,
        "translateY": 50
    }
}

{% endhighlight %}
</div>
The differences here are in <span class="code">imageSource</span> and <span class="code">imageSourceType</span> properties (Actually you can also see different values for <span class="code">tranlateX</span> and <span class="code">translateY</span> if you changed focal point in the UI): <span class="code">imageSource</span> contains server relative URL of the image that is used as a banner and <span class="code">imageSourceType</span> contains 2 instead of 4.<br />This information is enough to add (or replace) your own banner image on any page.<br />First, you need to add the picture to some location on your tenant - SharePoint or OneDrive.<br />Second, change <span class="code">imageSource</span> and <span class="code">imageSourceType</span> properties as shown below: <br />
<div markdown="1">
{% highlight javascript %}
var ctx = SP.ClientContext.get_current(); // SP context
var web = ctx.get_web(); // current web
var list = web.get_lists().getByTitle('Site Pages'); // pages library
var items = list.getItems(SP.CamlQuery.createAllItemsQuery()); // all items
ctx.load(items); // loading items from server...
ctx.executeQueryAsync(function() {
    var item = items.get_item(0); // here I'm working with single item by index. But you can iterate through all pages here
    var layoutWebpartsContent = item.get_item('LayoutWebpartsContent'); // getting content
    console.log(layoutWebpartsContent); // let's display the content
    
    var dataAttrContent = /data-sp-controldata="([^"]+)"/gmi.exec(layoutWebpartsContent); // getting data-sp-controldata content
    
    if (dataAttrContent.length) { 
        // we found the attribute.
        // Let's unescape and parse it to JSON
        // the content of the attribute is a 'group' in RegExp result. It will be located as second entry (with index 1)
        var unescaped = unescape(dataAttrContent[1]) // unescape
        var content = JSON.parse(unescaped.replace(/&amp;#58;/gmi, ':')); // replace &amp;#58; with :

        //
        // changing imageSource
        //
        if (!content.serverProcessedContent) {
            content.serverProcessedContent = {};
        }
        if (!content.serverProcessedContent.imageSources) {
            content.serverProcessedContent.imageSources = {};
        }
        content.serverProcessedContent.imageSources.imageSource = '/sites/contoso/SiteAssets/SitePages/banner.png';

        //
        // Changing imageSourceType
        //
        if (!content.properties) {
            content.properties = {};
        }
        content.properties.imageSourceType = 2;

        //
        // escaping back and updating item
        //
        debugger;
        var newContent = JSON.stringify(content);
        newContent = escape(newContent); // escaping
        newContent = newContent.replace(/%3A/gmi, ':').replace(/%2C/gmi, ','); // we need to replace %3A (:) with &amp;#58; and %2C (,) with ,
        layoutWebpartsContent = layoutWebpartsContent.replace(dataAttrContent[1], newContent);
        item.set_item('LayoutWebpartsContent', layoutWebpartsContent);
        item.update();
        ctx.executeQueryAsync(function() {
            console.log('success');
        }, function() {
            console.log('fail');
        });
    }
});

{% endhighlight %}
</div>
That's it.<br />Now your page should have the banner you wanted.<br />Have fun!