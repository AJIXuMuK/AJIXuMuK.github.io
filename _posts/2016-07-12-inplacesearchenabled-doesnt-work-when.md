---
layout: post
title: InplaceSearchEnabled doesn't work when set programmatically in XsltListViewWebPart
date: '2016-07-12T16:48:00.000-07:00'
author: Alex Terentiev
tags:
- JavaScript
- SharePoint 2013
- CSR
- C#
- Render Templates
- XsltListViewWebPart
- InplaceSearchEnabled
- JSLink
modified_time: '2016-07-13T08:45:12.345-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-6829899317489171942
blogger_orig_url: http://blog.aterentiev.com/2016/07/inplacesearchenabled-doesnt-work-when.html
---

There are a lot of forum threads about the error that InplaceSearchEnabled property of XsltListViewWebPart (XLV) cannot be changed programmatically, only through UI and WebPart properties.<br />I experienced the same issue when was trying to add XLV to sub site (display items of root level list on subsite) - the search input was not displayed and the InplaceSearchEnabled property was set to null.<br />Finally I found the solution (actually a workaround) and it is described below.<br /><br /><a name='more'></a>First I was trying to debug SharePoint.dll assembly (thanks to .NET Reflector) but the code was to complicated and full of Reflection. So I decided to look into the client-side part of the search functionality.<br />And here what I found:<br /><br /><ol><li>Search input is located inside <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">inplaceSearchDiv_&lt;wpq&gt;</span>&nbsp;div which is added to HTML in sp.ui.listsearchboxbootstrap.js. So we need to include this script to our page.</li><li>The div is rendered for the web part that is registered in <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">g_listSearchBoxInfo</span> global array and if ListSchema's property&nbsp;<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">InplaceSearchEnabled</span>&nbsp;is set to true.</li><li>The content of that div is added in sp.ui.listsearchbox.js.</li><li>In the code of the page with working search&nbsp;sp.ui.listsearchboxbootstrap.js is added as simple script,&nbsp;sp.ui.listsearchbox.js is registered as SOD with a bunch of dependencies:</li></ol>
<div markdown="1">
{% highlight html %}
<script src="/_layouts/15/sp.ui.listsearchboxbootstrap.js?rev=XoxCCd1ZTetlKjaD1x2A7A%3D%3D" type="text/javascript"></script>
<script type="text/javascript">RegisterSod("clientrenderer.js", "\u002f_layouts\u002f15\u002fclientrenderer.js?rev=PWwV4FATEiOxN90BeB5Hzw\u00253D\u00253D");</script>
<script type="text/javascript">RegisterSod("srch.resources.resx", "\u002f_layouts\u002f15\u002fScriptResx.ashx?culture=en\u00252Dus\u0026name=Srch\u00252EResources\u0026rev=T\u00252BEc8BFdsTQbQuVAobKatQ\u00253D\u00253D");</script>
<script type="text/javascript">RegisterSod("search.clientcontrols.js", "\u002f_layouts\u002f15\u002fsearch.clientcontrols.js?rev=5P1PlD9u0nLal8ZgtuGr6w\u00253D\u00253D");RegisterSodDep("search.clientcontrols.js", "clientrenderer.js");RegisterSodDep("search.clientcontrols.js", "srch.resources.resx");</script>
<script type="text/javascript">RegisterSod("profilebrowserscriptres.resx", "\u002f_layouts\u002f15\u002fScriptResx.ashx?culture=en\u00252Dus\u0026name=ProfileBrowserScriptRes\u0026rev=J5HzNnB\u00252FO1Id\u00252FGI18rpRcw\u00253D\u00253D");</script>
<script type="text/javascript">RegisterSod("sp.ui.listsearchbox.js", "\u002f_layouts\u002f15\u002fsp.ui.listsearchbox.js?rev=wws9SqyBBuflNZ2j2YOIAA\u00253D\u00253D");RegisterSodDep("sp.ui.listsearchbox.js", "search.clientcontrols.js");RegisterSodDep("sp.ui.listsearchbox.js", "profilebrowserscriptres.resx");</script>

{% endhighlight %}
</div>
So based on these observations I decided to add search using JavaScript (even if the InplaceSearchEnabled property itself was not set).<br />Here are the actions to initialize all we need:<br /><br /><ol><li>Add JSLink property to Web Part to load custom code when the Web Part is rendering</li><li>Load all needed scripts and register them as SOD if needed</li><li>Get Web Part's wpq identifier and register it in&nbsp;<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">g_listSearchBoxInfo</span>&nbsp;</li><li>Set <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ListSchema.InplaceSearchEnabled</span>&nbsp;to true</li><li>Enjoy the result</li></ol>And here is the code: <br />
<div markdown="1">
{% highlight javascript %}
/// we need to wait until clienttemplates.js is loaded
SP.SOD.executeFunc('clienttemplates.js', 'SPClientTemplates', function () {
    // code that will be run before rendering the script. It'a good time to configure everything for search input
    function onPreRender(ctx) {
        if (!window.g_listSearchBoxInfo)
            window.g_listSearchBoxInfo = [];
        var hasSearch = false;
        // if the searci is configured for this web part we won't want to add it twice
        for (var i = 0, len = window.g_listSearchBoxInfo.length; i < len; i++) {
            if (window.g_listSearchBoxInfo[i].wpq === ctx.wpq) {
                hasSearch = true;
                break;
            }
        }

        if (!hasSearch) {
            // the structure of the object was got during debugging session
            window.g_listSearchBoxInfo.push({
                fullSearchSiteUrl: _spPageContextInfo.webAbsoluteUrl,
                loadInProgress: false,
                searchBoxConstructor: null,
                sodFunc: null,
                sodKey: null,
                wpq: ctx.wpq
            });

            // setting InplaceSearchEnabled in ListSchema
            ctx.ListSchema.InplaceSearchEnabled = true;
        }
    }

    // registering CSR template override
    var overrideCtx = {};
    overrideCtx.OnPreRender = onPreRender;

    SPClientTemplates.TemplateManager.RegisterTemplateOverrides(overrideCtx);
    
    // adding scripts (with condition if there were previously added or not)
    var searchScript = jQuery('script[src*="listsearchboxbootstrap.js"]');
    if (!searchScript.length) {
        RegisterSod('clientrenderer.js', '/_layouts/15/clientrenderer.js');
        RegisterSod('srch.resources.resx', '/_layouts/15/ScriptResx.ashx?culture=en-us&amp;name=Srch.Resources');
        RegisterSod('search.clientcontrols.js', '/_layouts/15/search.clientcontrols.js')
        RegisterSodDep('search.clientcontrols.js', 'clientrenderer.js');
        RegisterSodDep('search.clientcontrols.js', 'srch.resources.resx');
        RegisterSod('profilebrowserscriptres.resx', '/_layouts/15/ScriptResx.ashx?culture=en-us&amp;name=ProfileBrowserScriptRes');
        RegisterSod('sp.ui.listsearchbox.js', '/_layouts/15/sp.ui.listsearchbox.js');
        RegisterSodDep('sp.ui.listsearchbox.js', 'search.clientcontrols.js');
        RegisterSodDep('sp.ui.listsearchbox.js', 'profilebrowserscriptres.resx');

        var script = document.createElement('script');
        script.type = 'text/javascript';
        script.src = '/_layouts/15/sp.ui.listsearchboxbootstrap.js';
        document.head.appendChild(script);
    }
});

{% endhighlight %}
</div>
This will add search input to the web part and it will work as expected.<br />Additional remarks: resources (.resx SODs) are loaded with culture parameter which is set to en-us here (I don't need any other culture). But if your portal will potentially use different cultures you can get the name of current culture from <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">_spPageContextInfo.</span>&nbsp;It will be available in onload event. The same I can say about /_layouts/15/ path. You can use&nbsp;<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">_spPageContextInfo.layoutsUrl</span>&nbsp;to get Layouts folder Url.<br />This code shows you how to <b>enable </b>the search. If it is enabled and you want to disable it you can just set&nbsp;<span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ListSchema.InplaceSearchEnabled</span>&nbsp;to false and remove registration of the wpq from <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">g_listSearchBoxInfo.</span><br /><br />I believe that's it.<br />Have fun!