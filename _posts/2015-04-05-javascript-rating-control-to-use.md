---
layout: post
title: JavaScript Rating control to use outside list views
date: '2015-04-05T13:06:00.000-07:00'
author: Alex Terentiev
tags:
- SharePoint Online
- SharePoint UI
- Rating
- SharePoint 2010
- Reputation
- SharePoint
- JavaScript
- SharePoint 2013
- executeQueryAsync
- Client Side Object Model
- CSOM
- ReputationModel
modified_time: '2015-04-05T13:12:15.913-07:00'
blogger_id: tag:blogger.com,1999:blog-3066084330774405472.post-3408836718497044240
blogger_orig_url: http://blog.aterentiev.com/2015/04/javascript-rating-control-to-use.html
---

There are a lot of tasks when you have to display information from list item in web part or on the page without list view.<br />For example, you have to create a web part to display any kind of media on the page. And this media contains a Rating field.<br />For such situations I've created a JavaScript control to render Rating Field outside list views. The control is based on&nbsp;<a href="https://rateit.codeplex.com/" target="_blank">RateIt</a>&nbsp;jQuery plugin.<br />So no more words, let's have some code.<br /><br /><a name='more'></a>I'm assuming that jQuery has been already loaded to the page.<br />
<div markdown="1">
{% highlight javascript %}
(function () {
    if (!window.YourNamespace)
        window.YourNamespace = {};
    var ns = window.YourNamespace;
    
    //
    // I want to be sure that scripts are loaded
    //
    SP.SOD.executeFunc('sp.js', 'SP.ClientContext', function () { });
    SP.SOD.executeOrDelayUntilScriptLoaded(function () { }, 'sp.js');

    SP.SOD.executeFunc('reputation.js', 'Microsoft.Office.Server.ReputationModel.Reputation', function () { });
    SP.SOD.executeOrDelayUntilScriptLoaded(function () { }, 'reputation.js');

    ns.RatingControl = function (settings) {
        if (!settings)
            return;
        var control = {
            parent: settings.parent, // parent DOM node (it could be just any css selector
            listId: settings.listId, // list ID
            itemId: settings.itemId, // item ID
            spContext: null,
            spWeb: null,
            spList: null,
            averageRating: null,
            ratingCount: null,
            _loadCss: function (src) {
                var css = document.createElement("link");
                css.setAttribute("rel", "stylesheet");
                css.setAttribute("type", "text/css");
                css.setAttribute("href", src);

                (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(css);
            },

            init: function () {
                var path = _spPageContextInfo.siteServerRelativeUrl + 'path_to_your_scripts';
                this._loadCss(path + 'rateit.css'); // load standard css for RateIt control
                this._loadCss(path + 'your-rating-control.css'); // load customizations
                jQuery.when(
                    jQuery.getScript(path + 'jquery.rateit.min.js'), // load RateIt
                    this.loadRatingInfo(), // load 
                    jQuery.Deferred(function (deferred) { deferred.resolve(); })
                ).done(jQuery.proxy(this.render, this));
            },

            loadRatingInfo: function () {
                var deferred = new jQuery.Deferred();

                if (!this.spContext) {
                    this.spContext = SP.ClientContext.get_current(); // getting Client Context
                    this.spWeb = this.spContext.get_web(); // getting Web
                    this.spList = this.spWeb.get_lists().getById(this.listId); // getting List
                }

                var item = this.spList.getItemById(this.itemId); // getting Item
                this.spContext.load(this.spWeb);
                this.spContext.load(this.spList);
                this.spContext.load(item); // here we can add the second parameter like 'Include(AverageRating, RatingCount)' 
                                           // to load only needed fields
                this.spContext.executeQueryAsync(jQuery.proxy(function () {
                    this.averageRating = item.get_item('AverageRating') || 0;
                    this.ratingCount = item.get_item('RatingCount') || 0;
                    deferred.resolve();
                }, this), function () {
                    deferred.resolve();
                });

                return deferred;
            },

            render: function () {
                var $parent = jQuery(this.parent),
                    $container = this.$container = jQuery('<div class="your-rating-control"></div>'); // initializing container

                $parent.append($container);
                // initializing RateIt control
                $container.rateit({
                    min: 0,  // minimum 0 stars
                    max: 5,  // maximum 5 stars
                    step: 1, // step is 1 star
                    resetable: false, // we  don't need reset button
                    value: this.averageRating, // current value
                    starwidth: 16, // width of the star image
                    starheight: 16 // height of the star image
                }).bind('rated', jQuery.proxy(this.onRated, this)); // bind to event
            },

            onRated: function (e, value) {
                // setting rating of the current user and gettng average one
                var rating = Microsoft.Office.Server.ReputationModel.Reputation.setRating(this.spContext,
                    this.listId, this.itemId, value);

                this.spContext.executeQueryAsync(jQuery.proxy(function () {
                    this.$container.rateit('value', rating.m_value); // setting average rating value to control
                }, this), function () {
                });
            }
        };

        return control;
    };
})();

{% endhighlight %}
</div>
<br />To modify the stars images override three css classes:<br />
<div markdown="1">
{% highlight css %}
div.rateit-range
{
    background: url('empty-star.png');
}

div.rateit-hover
{
    background: url('filled-star.png');
}

div.rateit-selected
{
    background: url('filled-star.png');
}

{% endhighlight %}
</div>
<br />Now you can use it:<br />
<div markdown="1">
{% highlight javascript %}
var control = new YourNamespace.RatingControl({
    parent: '#some-div',
    listId: 'guid-string',
    itemId: 'intId'
});
control.init();

{% endhighlight %}
</div>
<br />Have fun!