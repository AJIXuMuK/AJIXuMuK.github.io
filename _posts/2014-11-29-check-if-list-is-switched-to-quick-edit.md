---
layout: post
title: Check if list is switched to quick edit mode
date: '2014-11-29T19:51:00.000-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint UI
- SharePoint 2013
- SharePoint 2010
- SharePoint
modified_time: '2015-02-11T14:14:16.930-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6237662530007626048
blogger_orig_url: http://blog.aterentiev.com/2014/11/check-if-list-is-switched-to-quick-edit.html
---

<div dir="ltr" style="text-align: left;" trbidi="on">In the last project I was working on, I've stucked in the problem that SharePoint Client Side Object Model and javascript variables do not contain any information about the list mode: view or quick edit. And Google keeps silence about the way to get it.<br />So I've analyzed a mark-up and here are some ways to handle the switch to quick edit mode.<br /><br /><a name='more'></a>First thing that I've tried was jquery selectors to find the "edit" link. I don't like this method because you can't sure that nobody will add an element on the page that will have the same class, attribute or even id. But it the case this approach doesn't work anyway because there is no any specific class or attribute you can use to find the elements by selector correctly.<br />Then I've dug a little bit deeper. The onclick handler for the "edit" link looks like that: <br />

<div markdown="1">
{% highlight javascript %}
onclick="EnsureScriptParams('inplview', 'InitGridFromView', '{5C96BEED-04A9-4A4B-AF91-FA3EEE1C2C76}'); return false;"
{% endhighlight %}
</div>

So maybe I can just change the <span style="font-family: Courier New, Courier, monospace;">EnsureScriptParams</span>&nbsp;function to add some custom code (Thanks to&nbsp;<a href="http://en.wikipedia.org/wiki/Brendan_Eich" target="_blank">Brendan Eich</a>&nbsp;that we can do that in javascript). But unfortunately this function is used in many cases to ensure that some script is loaded and the function exists.<br />In case of "Edit" link it used to ensure <span style="font-family: Courier New, Courier, monospace;">inplview</span>&nbsp;script and <span style="font-family: Courier New, Courier, monospace;">InitGridFromView</span>&nbsp;function.<br />This is it! The solution is to change the <span style="font-family: Courier New, Courier, monospace;">InitGridFromView</span>. This function will be called when any list on the page is switched to the quick edit mode.<br />So the result code is simple (one more thanks to Brendan): <br />
<div markdown="1">
{% highlight javascript %}
//
// We have to be sure that the script is loaded
//
SP.SOD.executeFunc('inplview.js', 'InitGridFromView', function () { });
SP.SOD.executeOrDelayUntilScriptLoaded(function () {

    if (window.InitGridFromView) {
        var baseInitGridFromView = window.InitGridFromView;
        window.InitGridFromView = function (a, c) {
            baseInitGridFromView(a, c);
            console.log('hello from grid init');
        };
    }
}, 'inplview.js');

{% endhighlight %}
</div>
If you know any other and more simple way to track the mode change please leave a comment.<br />Have fun!</div>