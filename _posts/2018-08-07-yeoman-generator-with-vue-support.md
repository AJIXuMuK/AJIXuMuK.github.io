---
layout: post
title: 'Yeoman Generator With Vue Support Update: Extensions Support!'
date: '2018-08-07T19:09:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- Vuejs
- SharePoint Framework
- Office 365
- SharePoint Framework Extensions
- SharePoint
- Client Side Web Part
- SPFx
- Vue
- Yeoman
- generator-sharepoint
- O365
modified_time: '2018-10-06T12:58:54.274-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-8401931003611477209
blogger_orig_url: http://blog.aterentiev.com/2018/08/yeoman-generator-with-vue-support.html
---

<b>UPDATE: the generator has been included into <a href="https://github.com/pnp/generator-spfx/" target="_blank">@pnp/spfx</a> generator. Please use this one for the latest version.</b><br /><br />In <a href="http://blog.aterentiev.com/using-vuejs-in-sharepoint-framework">Using Vue.js in SharePoint Framework Applications. Part III: Yeoman Generator with Vue Support</a> I announced that I've published a Yeoman generator for SharePoint Framework solutions with Vue.js support.<br />The initial version of the generator supported SPFx web parts only. There was no way to create SPFx extensions using the generator.<br />Today I happy to announce that Extensions support has been added to the generator!<br />Now you can select any of the available types of SPFx projects.<br />NPM package: <a href="https://www.npmjs.com/package/generator-vuespfx">generator-vuespfx</a><br />GitHub repo: <a href="https://github.com/AJIXuMuK/generator-vuespfx">https://github.com/AJIXuMuK/generator-vuespfx</a><br />Wiki: <a href="https://github.com/AJIXuMuK/generator-vuespfx/wiki">Wiki</a><br /><a name='more'></a><h2>How to use</h2>To use it just install the package to your machine: 
<div markdown="1">
{% highlight console %}

npm i -g generator-vuespfx

{% endhighlight %}
</div>
And then use it in the same way as <span class="code">@microsoft/generator-sharepoint</span>: 
<div markdown="1">
{% highlight console %}

yo vuespfx

{% endhighlight %}
</div>
This will: <ul><li>create SPFx solution with selected type of SharePoint Framework component,</li>  <li>add all needed dependencies for Vue.js,</li><li><b>for Client-Side Web Parts and Field Customizers</b><ul><li>create Vue.js Single File Component (SFC) with default markup</li><li>add SFC component to the web part's/field customizer's code</li></ul></li></ul><br />Please, leave your feedback and contribute to make the generator even better! =) <br /><br />That's it!<br />Have fun!