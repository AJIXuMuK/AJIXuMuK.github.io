---
layout: post
title: Logically Structure Localizable Strings in SharePoint Framework Solutions
date: '2018-03-05T20:23:00.000-08:00'
author: Alex Terentiev
tags:
- SharePoint Online
- i18
- SharePoint 2016
- SPFx
- Localization
- O365
- SharePoint Framework
- SharePoint
modified_time: '2018-03-06T10:46:43.664-08:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-7131562158604064801
blogger_orig_url: http://blog.aterentiev.com/2018/03/logically-structure-localizable-strings.html
---

In large projects there are a lot of texts that should be localized. And it might be a mess to manage them all if they are just plainly added in a single resources file.<br />That's why I want to discuss how we can logically structure resources (strings) in SPFx solutions.<br /><br /><a name='more'></a><br /><h2>Out of the box</h2>First, let's look at what SPFx solution proposes in terms of localization.<br />The localization files used by the solution are stored in the <b>./src/&lt;webparts|extensions&gt;/&lt;name-of-solution&gt;/loc</b> folder.<br />This folder contains a TypeScript type definition file (<b>loc/mystrings.d.ts</b>) that informs TypeScript of the different strings included in the localized files. Using the information from this file, your code editor can provide you with IntelliSense when working with strings in code. Additionally, while building your project, TypeScript can verify that you're not referring to a string that hasn't been defined. And you can imagine how this file will look if the solution contains a lot of strings.<br />For each locale supported by your web part, there is also a plain JavaScript file (not TypeScript) named in lowercase after the locale (for example <b>en-us.js</b>) that contains the translated strings.<br />Mapping between specific module definition from TypeScript file and translated strings is done in <b>config/config.json</b> file in <b>localizedResources</b> section: 
<div markdown="1">
{% highlight javascript %}

"localizedResources": {
    "YourSolutionStrings": "lib/webparts|extensions/yoursolution/loc/{locale}.js"
  }

{% endhighlight %}
</div>
And usage of the localized resources in the code is as simple as importing the module and referencing specific property of the interface: 
<div markdown="1">
{% highlight typescript %}

import * as strings from YourSolutionStrings;
//...
<Dialog
  title={strings.DialogTitle}>

{% endhighlight %}
</div>
<br /><h2>Option 1: Nested objects in a single localization file</h2>This option allows you to use single localization file for all the strings. Disadvantage of the approach is that IntelliSense feature will be lost and TypeScript compiler will not check second level properties' names and won't inform about misspelling. It may lead to <span class="style">undefined</span> texts in the solution.<br />To use nested objects do the following: <ul><li>In type definition file (<b>.d.ts</b>) add property declaration with <span class="style">object</span> type: 
<div markdown="1">
{% highlight typescript %}

AboutDialog: { [key: string]: string };

{% endhighlight %}
</div>
</li><li>In locale JavaScript file describe all the properties for the object: 
<div markdown="1">
{% highlight javascript %}

"AboutDialog": {
    "Title": "About",
    "Edition": "Enterprise"
}

{% endhighlight %}
</div>
</li><li>use strings in the code: 
<div markdown="1">
{% highlight typescript %}

<Dialog
    title={strings.AboutDialog['Title']}>
    <span>{strings.AboutDialog['Edition']}</span>
</Dialog>

{% endhighlight %}
</div>
</li></ul><br /><h2>Option 2: Multiple localization files</h2>This option requires to create separate files for different logical parts of the solutions. The good thing is that developer doesn't loose IntelliSense and misspelling errors notifications. The disadvantage is necessity to make separate request for each file in runtime.<br />To use multiple localization files: <ul><li>Add as much type definition files and local JavaScript files as needed. For example, <b>aboutDialog.d.ts</b> and <b>aboutDialog.en-us.js</b></li><li>dd strings to the files in the same manner as it is done for the default localization file in the project. Don't forget to use unique interface and module name: 
<div markdown="1">
{% highlight typescript %}

// d.ts
declare interface IAboutDialogStrings {
    Title: string;
    Edition: string;
  }
  
  declare module 'AboutDialogStrings' {
    const strings: IAboutDialogStrings;
    export = strings;
  }

  // en-us.js
  define([], function () {
    return {
        "Title": "About",
        "Edition": "Enterprise"
    }
});

{% endhighlight %}
</div>
</li><li>Reference new localization files in <b>config/config.json</b>: 
<div markdown="1">
{% highlight javascript %}

"localizedResources": {
  "AboutDialogStrings": "lib/webparts/yourwebpart/loc/aboutDialog.{locale}.js"
}

{% endhighlight %}
</div>
</li><li>Use new file (module) in the code: 
<div markdown="1">
{% highlight typescript %}

import * as aboutStrings from 'AboutDialogStrigs';
// ...
<Dialog
    title={aboutStrigs.Title}>
    <span>{aboutStrings.Edition}</span>
</Dialog>

{% endhighlight %}
</div>
</li></ul><br /><h2>Instead of conclusion</h2>You can use any of these two approaches or combine them.<br />When selecting which one to use take into consideration the cons of the approaches.<br />Also, it doesn't make sense to complicate your life if you have few strings in the localization file ;)<br />You can find an example project that uses both approaches here: <a href="https://github.com/AJIXuMuK/SPFx/tree/master/strings" target="_blank">https://github.com/AJIXuMuK/SPFx/tree/master/strings</a><br /><br />Have fun!