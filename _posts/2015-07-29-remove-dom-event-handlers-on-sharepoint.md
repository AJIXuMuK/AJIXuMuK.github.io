---
layout: post
title: Remove DOM event handlers on SharePoint Page
date: '2015-07-29T16:16:00.000-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SharePoint UI
- SharePoint 2013
- Sys.UI.DomEvent
- SharePoint
modified_time: '2015-07-29T16:18:17.907-07:00'
thumbnail: http://4.bp.blogspot.com/-QoFkppx4K5I/Vbldh_lnz1I/AAAAAAAAAT4/NLxHe8V64e8/s72-c/Untitled-1.png
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6057128570468897620
blogger_orig_url: http://blog.aterentiev.com/2015/07/remove-dom-event-handlers-on-sharepoint.html
---

I won't write a lot of words in here...<br /><b>Environment: </b>SP Online, Publishing Portal<br /><b>Problem: </b><span style="font-family: Courier New, Courier, monospace;">jQuery.off()</span>, <span style="font-family: Courier New, Courier, monospace;">jQuery.unbind(),</span> <span style="font-family: Courier New, Courier, monospace;">element.removeEventHandler</span> didn't work so we need to find a way to remove event handlers.<br /><b>Solution:</b><br /><a name='more'></a><br />SharePoint uses&nbsp;<a href="https://msdn.microsoft.com/en-us/library/vstudio/bb310798(v=vs.100).aspx" target="_blank"><span style="font-family: Courier New, Courier, monospace;">Sys.UI.DomEvent.addHandler</span></a> (alias <span style="font-family: Courier New, Courier, monospace;">$addHandler</span>) and <a href="https://msdn.microsoft.com/en-us/library/vstudio/bb310811(v=vs.100).aspx" target="_blank"><span style="font-family: Courier New, Courier, monospace;">Sys.UI.DomEvent.addHandlers</span></a> (alias <span style="font-family: Courier New, Courier, monospace;">$addHandlers</span>) to add DOM events handlers.<br />If you want to delete absolutely all handler for all events everything is simple - just call <a href="https://msdn.microsoft.com/en-us/library/vstudio/bb310816(v=vs.100).aspx" target="_blank"><span style="font-family: Courier New, Courier, monospace;">Sys.UI.DomEvent.clearHandlers</span></a> and put your DOM element as a parameter.<br />But what if you want to delete handlers of particular event?..<br />In that case you can use&nbsp;<a href="https://msdn.microsoft.com/en-us/library/vstudio/bb397510(v=vs.100).aspx" target="_blank"><span style="font-family: Courier New, Courier, monospace;">Sys.UI.DomEvent.removeHandler</span></a> (alias <span style="font-family: Courier New, Courier, monospace;">$removeHandler</span>) to remove added handlers... But the thing is that you need to know the handlers' functions to use this method.<br />After some investigation I've found that added handlers are stored in _events attribute in DOM element:<br /><div class="separator" style="clear: both; text-align: center;"><img border="0" height="151" src="{{site.baseurl}}/assets/images/posts/2015/2015-07-29.png" width="320" /></div>The functions that we need to use <span style="font-family: Courier New, Courier, monospace;">$removeHandler </span>are stored in handler property of each item in the array of particular event handlers.<br />And here is an example of code to use to remove all 'click' handlers from 'form' element:<br /><br />
<div markdown="1">
{% highlight javascript %}
var $form = jQuery('form');

if ($form.length) {
  var formEvents = $form[0]._events;

  if (formEvents &amp;&amp; formEvents.click) {
    jQuery.each(formEvents.click, function (clickIndex, clickHandler) {
      if (clickHandler.handler)
        $removeHandler($form[0], 'click', clickHandler.handler);
    });
  }
}

{% endhighlight %}
</div>
<br />Have Fun!