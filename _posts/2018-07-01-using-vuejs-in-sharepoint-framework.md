---
layout: post
title: 'Using Vue.js in SharePoint Framework Applications. Part III: Yeoman Generator with Vue Support'
date: '2018-07-01T16:19:00.000-07:00'
author: Alex Terentiev
tags:
  - SharePoint Online
  - Client Side Web Part
  - SPFx
  - Vuejs
  - Vue
  - Yeoman
  - generator-sharepoint
  - O365
  - SharePoint Framework
  - Office 365
  - SharePoint
modified_time: '2018-10-06T12:58:34.465-07:00'
blogger_id: 'tag:blogger.com,1999:blog-3066084330774405472.post-6765945008111191756'
blogger_orig_url: 'http://blog.aterentiev.com/2018/07/using-vuejs-in-sharepoint-framework.html'
slug: vue-js-spfx-yeoman-generator
---

<b>UPDATE: the generator has been included into <a href="https://github.com/pnp/generator-spfx/" target="_blank">@pnp/spfx</a> generator. Please use this one for the latest version.</b><br /><br />This is the third post about SharePoint Framework and Vue.js. In this post I want to announce a Yeoman generator for SPFx web part with Vue.js support.<br /><br />List of posts:<br /><ol><li><a href="/vue-js-spfx-whats-whys" target="_blank">Whats and Whys</a></li><li><a href="/vue-js-spfx-default-web-part" target="_blank">Default SPFx web part using Vue.js</a></li><li>Yeoman generator with Vue support (this post)</li><li><a href="/vue-js-spfx-prop-pane-control">Web Part Property Pane Control</a></li><li><a href="/vue-js-spfx-react-in-vue-js-solution" target="_blank">Use React Components inside Vue.js Solution</a></li></ol><a name='more'></a>So, in the previous post we went through all the needed configurations to add Vue.js support to SharePoint Framework project and to implement simple web part using Vue Single File Component.<br />And not it feels natural that it would be great to have automatic way of creating SPFx projects with Vue.js support.<br />That's why I've created a Yeoman generator that does all the stuff automatically.<br />The generator in called <span class="code">vuespfx</span> and available as <a href="https://www.npmjs.com/package/generator-vuespfx" target="_blank">NPM package</a>.<br />To use it just install the package to your machine: 
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
This will <ul><li>create SPFx solution with a client-side web part,</li>  <li>add all needed dependencies for Vue.js,</li><li>create Vue.js Single File Component (SFC) with default web part markup</li><li>add SFC component to the web part's code</li></ul>Currently the generator supports web parts only. But I'll update it to add Extensions support as well.<br />If you want to contribute, feel free. Go to <a href="https://github.com/AJIXuMuK/generator-vuespfx" target="_blank">https://github.com/AJIXuMuK/generator-vuespfx</a> fork it and create pull requests. <br />Please, leave your feedback regarding the generator! <br /><br />That's it!<br />Have fun and stay tuned!
