---
layout: post
title: Close Modal Dialog from App page in Provider-Hosted App
date: '2015-04-20T17:53:00.001-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint Online
- SP.UI.ModalDialog
- SharePoint 2013
- ASP.NET
- Provider-Hosted App
- postMessage
- CustomAction
- SharePoint Apps
modified_time: '2015-04-20T18:06:16.103-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-2782415257670146962
blogger_orig_url: http://blog.aterentiev.com/2015/04/close-modal-dialog-from-app-page-in.html
---

Sometimes there is a necessity to show some App page in Modal Dialog.<br />For example, show some settings on custom action click.<br />It is simple to show the dialog, but standard approach of closing it doesn't work because the App and SharePoint site can be hosted on different domains. And we can't just commit popup from cross-domain frame document.<br />To close the dialog you can use HTML5 postMessage API.<br /><br /><a name='more'></a>If we speak about some custom action, and, to be correct, about custom action that was created <b>declaratively</b>&nbsp;with <span style="font-family: Courier New, Courier, monospace;">HostWebDialog</span> property set to true, you can use messages <span style="font-family: Courier New, Courier, monospace;">CloseCustomActionDialogRefresh</span> to close the dialog and refresh the page or <span style="font-family: Courier New, Courier, monospace;">CloseCustomActionDialogNoRefresh</span> to close but without refresh:<br />
<div markdown="1">
{% highlight javascript %}
var target = parent.postMessage ? parent : 
    (parent.document.postMessage ? parent.document : undefined); 
if (target) 
    target.postMessage((doRefresh ? 'CloseCustomActionDialogRefresh' : 'CloseCustomActionDialogNoRefresh'), '*');

{% endhighlight %}
</div>
Server-side:<br />
<div markdown="1">
{% highlight csharp %}
private void CloseDialog()
{
    this.Context.Response.Write(
        "<script>" + // add type="text/javascript" here. It is removed to avoid syntax highlighter issues
        "    var target = parent.postMessage ? parent :" +
        "        (parent.document.postMessage ? parent.document : undefined);" +
        "    if (target)" +
        "        target.postMessage((doRefresh ? 'CloseCustomActionDialogRefresh' : 'CloseCustomActionDialogNoRefresh'), '*');" +
        "</script>");
    this.Context.Response.Flush();
    this.Context.Response.End();
}

{% endhighlight %}
</div>
If you open the dialog by custom code (code-created custom action has no HostWebDialog property so it is kind of a dialog opened by custom code) SharePoint doesn't listen to the messages described above. But you can use <span style="font-family: Courier New, Courier, monospace;">CloseDialog</span> message and <span style="font-family: Courier New, Courier, monospace;">dialogReturnValueCallback</span> to refresh the page:<br />
<div markdown="1">
{% highlight javascript %}
// open dialog with callback handler example:
SP.UI.ModalDialog.showModalDialog({ 
    url: 'your_app_page_url_or_some_proxy_page_url', 
    // ...
    dialogReturnValueCallback: function CallDETCustomDialog(dialogResult, returnValue) { SP.UI.ModalDialog.RefreshPage(SP.UI.DialogResult.OK); } });

// posting a message
var target = parent.postMessage ? parent : 
    (parent.document.postMessage ? parent.document : undefined); 
if (target) 
    target.postMessage('CloseDialog', '*');

{% endhighlight %}
</div>
Server-side:<br />
<div markdown="1">
{% highlight csharp %}
private void CloseDialog()
{
    this.Context.Response.Write(
        "<script>" + // add type="text/javascript" here. It is removed to avoid syntax highlighter issues
        "    var target = parent.postMessage ? parent :" +
        "        (parent.document.postMessage ? parent.document : undefined);" +
        "    if (target)" +
        "        target.postMessage('CloseDialog', '*');" +
        "</script>");
    this.Context.Response.Flush();
    this.Context.Response.End();
}

{% endhighlight %}
</div>
<br />That's all, have fun!<br /><br />