---
layout: post
title: Using SharePoint Property Bags in javascript code
date: '2014-11-27T15:02:00.000-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint UI
- SharePoint 2013
- SharePoint 2010
- Client Side Object Model
- CSOM
- Property Bag
- SharePoint
modified_time: '2014-11-29T19:53:55.338-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-1107737807378902070
blogger_orig_url: http://blog.aterentiev.com/2014/11/using-sharepoint-property-bags-in.html
---

<div dir="ltr" style="text-align: left;" trbidi="on"><br /><div class="p1"><span class="s1">SharePoint Property Bag allows to store configurations settings at different levels of the SharePoint hierarchy outside of the&nbsp;application itself. Property bag is basically a hash table of key-value pair options.</span></div><div class="p1"><span class="s1">I'll show how to use Property Bag of a <span style="font-family: Courier New, Courier, monospace;">SPWeb</span>&nbsp;object.</span><br /><a name='more'></a></div><div class="p1"><span class="s1">First of all you have to be sure that sp.js script is loaded. Usually&nbsp;<a href="http://msdn.microsoft.com/en-us/library/office/ff411788%28v=office.14%29.aspx" target="_blank">SP.SOD.executeOrDelayUntilScriptLoaded</a>&nbsp;<span class="s1">is used to execute custom code after script is loaded. But sometimes (I don't really know why) it doesn't work. That's why you can insert a call of&nbsp;<a href="http://msdn.microsoft.com/en-us/library/office/ff409592%28v=office.14%29.aspx" target="_blank">SP.SOD.executeFunc</a>&nbsp;fucntion before&nbsp;</span></span><span class="s1">executeOrDelayUntilScriptLoaded. And that will work in any case:</span></div>
<div markdown="1">
{% highlight javascript %}
// we need both calls because in some situations executeOrDelayUntilScriptLoaded doesn't work
SP.SOD.executeFunc('sp.js', 'SP.ClientContext', function () { });
SP.SOD.executeOrDelayUntilScriptLoaded(function () {
    // code goes here
}, 'sp.js');

{% endhighlight %}
</div>
<div class="p1"><span class="s1">After the script is loaded you can access </span><span style="font-family: Courier New, Courier, monospace;">currentContext</span><span class="s1">&nbsp;object and get </span><span style="font-family: Courier New, Courier, monospace;">web</span><span class="s1">&nbsp;to work with:</span><br />
<div markdown="1">
{% highlight javascript %}
var spContext = SP.ClientContext.get_current(),
spWeb = spContext.get_web();

{% endhighlight %}
</div>
<span class="s1">Actually you have no </span><span style="font-family: Courier New, Courier, monospace;">web</span><span class="s1">&nbsp;object before you call </span><span style="font-family: Courier New, Courier, monospace;"><a href="http://msdn.microsoft.com/en-us/library/office/dn168903%28v=office.15%29.aspx" target="_blank">load</a></span><span class="s1">&nbsp;and </span><span style="font-family: Courier New, Courier, monospace;"><a href="http://msdn.microsoft.com/en-us/library/office/dn168907%28v=office.15%29.aspx" target="_blank">executeQueryAsync</a></span><span class="s1">&nbsp;functions of </span><span style="font-family: Courier New, Courier, monospace;">currentContext</span><span class="s1">&nbsp;instance (you can read <a href="http://msdn.microsoft.com/en-us/library/office/jj163201(v=office.15).aspx" target="_blank">msdn post</a>&nbsp;for clarification).</span><br /><span class="s1">The first parameter of&nbsp;</span><span style="font-family: Courier New, Courier, monospace;">load</span><span class="s1">&nbsp;function is an object that contains the properties to retrieve from the server. Other parameters are optional and contain names of additional properties you want to retrieve.</span><br /><span class="s1">In our case you need to get <span style="font-family: Courier New, Courier, monospace;">AllProperties</span>&nbsp;property from the server:</span><br />
<div markdown="1">
{% highlight javascript %}
spContext.load(spWeb, 'AllProperties');
{% endhighlight %}
</div>
Actually, <span style="font-family: Courier New, Courier, monospace;">AllProperties</span><span class="s1">&nbsp;client object is a wrapping-object. That's why you need to call internal function to get key-value pairs of the Property Bag:</span><br />
<div markdown="1">
{% highlight javascript %}
 var allProps = spWeb.get_allProperties(),
// actually allProps.get_fieldValues() contains key-value pairs of the Property Bag
allPropsValues = allProps.get_fieldValues(),
// value of the specific key
value = allPropsValues[key];

{% endhighlight %}
</div>
<span class="s1">To set some property to the Property Bag you should write something like that:</span><br />
<div markdown="1">
{% highlight javascript %}
allProps.set_item(key, value);
spWeb.update();
spContext.executeQueryAsync(function () {
    // some code
}, function () {
    //some logging code
});

{% endhighlight %}
</div>
<span style="font-family: Courier New, Courier, monospace;">value</span><span class="s1">&nbsp;should be a string. Use </span><a href="https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify" style="font-family: 'Courier New', Courier, monospace;" target="_blank">JSON.stringify()</a><span class="s1">&nbsp;to convert json object to string and store it to the Property Bag (you can than use </span><span style="font-family: Courier New, Courier, monospace;"><a href="https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse" target="_blank">JSON.parse()</a></span><span class="s1">&nbsp;to restore the object).</span><br /><span class="s1">Full test code looks like that:</span><br />
<div markdown="1">
{% highlight javascript %}
SP.SOD.executeFunc('sp.js', 'SP.ClientContext', function () { });
SP.SOD.executeOrDelayUntilScriptLoaded(function () {
    var spContext = SP.ClientContext.get_current(),
    spWeb = spContext.get_web();
    spContext.load(spWeb, 'AllProperties');
    spContext.executeQueryAsync(function () {
        
        var allProps = spWeb.get_allProperties(),
        key = 'testKey',
        // actually allProps.get_fieldValues() contains key-value pairs of the Property Bag
        allPropsValues = allProps.get_fieldValues(),
        // value of the specific key
        value = allPropsValues[key];

        allProps.set_item(key, value);
        spWeb.update();
        spContext.executeQueryAsync(function () {
            // some code
        }, function () {
            //some logging code
        });
    });
}, 'sp.js');

{% endhighlight %}
</div>
I'll be glad to have comments and feedback.<br /><span class="s1">Have fun!&nbsp;</span></div></div>