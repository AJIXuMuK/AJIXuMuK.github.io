---
layout: post
title: Minimal Download Strategy and window.setTimeout issue
date: '2014-12-19T14:05:00.000-08:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint 2013
- setTimeout
- SharePoint
modified_time: '2014-12-19T14:06:09.105-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-8157128571039255578
blogger_orig_url: http://blog.aterentiev.com/2014/12/minimal-download-strategy-and.html
---

<div dir="ltr" style="text-align: left;" trbidi="on">Sometimes developers from Microsoft create mysterious things...<br />In two words the issue with setTimeout looks like that.<br /><a name='more'></a><br />There is a standard function <a href="http://www.w3schools.com/jsref/met_win_settimeout.asp" style="font-family: 'Courier New', Courier, monospace;" target="_blank">window.setTimeout</a>.&nbsp;You can provide the function to call, the interval in milliseconds to wait and some additional arguments to be applied to the function. For example:<br />
<div markdown="1">
{% highlight javascript %}
setTimeout(function(firstName, lastName) {
  alert(firstName + ' ' + lastName);
}, 1000, 'Alex', 'Terentiev');

{% endhighlight %}
</div>
And that code works fine if the Minimal Download Strategy is deactivated. But if it is activated you'll see "undefined undefined" in the alert window.<br />It is so because of the override in the start.js file:<br />
<div markdown="1">
{% highlight javascript %}
window.setTimeout = function() {
  var b;
  if (arguments.length > 1 &amp;&amp; "function" == typeof arguments[0]) {
    var c = arguments[0];
    arguments[0] = function() {
      c();
      if ("undefined" != typeof b) {
        var d = Array.indexOf(a._timeoutArray, b);
        -1 != d &amp;&amp; a._timeoutArray.splice(d, 1)
      }
    }
  }
  if ("function" == typeof e)
    b = e.apply(window, arguments);
  else
    b = e(arguments[0], arguments[1]);
  "undefined" != typeof b &amp;&amp; a._timeoutArray.push(b);
  return b
}

{% endhighlight %}
</div>
The function you're trying to call is stored in variable <span style="font-family: Courier New, Courier, monospace;">c</span>. And it is calling just like <span style="font-family: Courier New, Courier, monospace;">c()</span> which means that it will not have any arguments you were passing to setTimeout.<br />It is just awesome!<br />You can fix that issue by adding some code to the callee function (the function you're calling after timeout): <br />
<div markdown="1">
{% highlight javascript %}
if (!arguments.length) {
  var caller = arguments.callee.caller;

  if (caller) {
    var callerArgs = caller.arguments;
    // get arguments from callerArgs
  }
}

{% endhighlight %}
</div>
Have fun!</div>